/* TypeScript (TS) @Armilar
 *
 * This Script will read the consumption Data from Daikin-Cloud adapter and transform the raw data into individual data points similar to how it's shown in the Daikin Onecta App.
 * It will also sum up the historic consumption from the current and previous year and update the total consumption meter going forward.
 * 
 * The data is created in the following path analogous to the daikin cloud data
 * 0_userdata.0.daikin-cloud.0...
 * 
 * Createtd: 14.11.2024
 */

// Log-Mode
const logMode: any = 'info'; // 'info' or 'debug'
// Path in 0_userdata
const mainPath: string = '0_userdata.0.';
// Onnecta hours (if d-raw)
const hourly: Array<string>  = ["00:00 - 02:00", "02:00 - 04:00", "04:00 - 06:00", "06:00 - 08:00", "08:00 - 10:00", "10:00 - 12:00", "12:00 - 14:00", "14:00 - 16:00", "16:00 - 18:00", "18:00 - 20:00", "20:00 - 22:00", "22:00 - 24:00"];
// Onnecta weekdays (if w-raw)
const daily: Array<string>   = ["01_Monday", "02_Tuesday", "03_Wednesday", "04_Thursday", "05_Friday", "06_Saturday", "07_Sunday"];
// Onnecta months  (if m-raw)
const monthly: Array<string> = ["01_Januray", "02_February", "03_March", "04_April", "05_May", "06_June", "07_July", "08_August", "09_September", "10_October", "11_November", "12_December"];

let devices: string[] = getDeviceNames("daikin-cloud.");

function setOrCreateState(id: string, value: any, forceCreation: boolean = true, common: Partial<iobJS.StateCommon> = {}, callback?: iobJS.SetStateCallback): void {
    if (!existsState(id)) {
        createState(id, value, forceCreation, common, callback);
    } else {
        setState(id, value, true);
    }
}

// Get Daikin Device Names
function getDeviceNames(vAdapterInstance: string): string[] {
    let devices = [];
    $(vAdapterInstance + '*raw').each(function (id) {
        if (devices.indexOf(id) == -1) {
            devices.push(id);
        }
    });
    return devices;
}

async function writeConsumtionData(path: string, rawType: string, rawData: number[]): Promise<void> {
    log(rawType + ' - ' + rawData, logMode);
    let total: number = 0;
    switch (rawType) {
	case "d-raw":
            for (let j = 0; j < 12; j++) {
                let dpName = "Yesterday." + hourly[j];
                setOrCreateState(mainPath + path + dpName, rawData[j], true, {type: 'number', name: hourly[j], role: 'value.power', unit: 'kWh'});
                log(mainPath + path + dpName + ': ' + rawData[j], logMode);
                total = total + rawData[j];
            }
            setOrCreateState(mainPath + path + 'Total.Yesterday', total, true, {type: 'number', name: 'Yesterday', role: 'value.power',unit: 'kWh'});
            total = 0;
            for (let j = 12; j < 24; j++) {
                let dpName = "Today." + hourly[j-12];
                setOrCreateState(mainPath + path + dpName, rawData[j], true, {type: 'number', name: hourly[j-12], role: 'value.power', unit: 'kWh'});
                log(mainPath + path + dpName + ': ' + rawData[j], logMode);
                total = total + rawData[j];
            }
            setOrCreateState(mainPath + path + 'Total.Today', total, true, {type: 'number', name: 'Today', role: 'value.power',unit: 'kWh'});
            break;
	case "w-raw":
            for (let j = 0; j < 7; j++) {
                let dpName = "LastWeek." + daily[j];
                setOrCreateState(mainPath + path + dpName, rawData[j], true, {type: 'number', name: daily[j], role: 'value.power', unit: 'kWh'});
                log(mainPath + path + dpName + ': ' + rawData[j], logMode);
                total = total + rawData[j];
            }
            setOrCreateState(mainPath + path + 'Total.LastWeek', total, true, {type: 'number', name: 'LastWeek', role: 'value.power',unit: 'kWh'});
            total = 0;
            for (let j = 7; j < 14; j++) {
                let dpName = "ThisWeek." + daily[j-7];
                setOrCreateState(mainPath + path + dpName, rawData[j], true, {type: 'number', name: daily[j-7], role: 'value.power', unit: 'kWh'});
                log(mainPath + path + dpName + ': ' + rawData[j], logMode);
                total = total + rawData[j];
            }
            setOrCreateState(mainPath + path + 'Total.ThisWeek', total, true, {type: 'number', name: 'ThisWeek', role: 'value.power',unit: 'kWh'});
            break;
	case "m-raw":
            for (let j = 0; j < 12; j++) {
                let dpName = "LastYear." + monthly[j];
                setOrCreateState(mainPath + path + dpName, rawData[j], true, {type: 'number', name: monthly[j], role: 'value.power', unit: 'kWh'});
                log(mainPath + path + dpName + ': ' + rawData[j], logMode);
                total = total + rawData[j];
            }
            setOrCreateState(mainPath + path + 'Total.LastYear', total, true, {type: 'number', name: 'LastYear', role: 'value.power',unit: 'kWh'});
            total = 0;
            for (let j = 12; j < 24; j++) {
                let dpName = "ThisYear." + monthly[j-12];
                setOrCreateState(mainPath + path + dpName, rawData[j], true, {type: 'number', name: monthly[j-12], role: 'value.power', unit: 'kWh'});
                log(mainPath + path + dpName + ': ' + rawData[j], logMode);
                total = total + rawData[j];
            }
            setOrCreateState(mainPath + path + 'Total.ThisYear', total, true, {type: 'number', name: 'ThisYear', role: 'value.power',unit: 'kWh'});
            break;
    }
}

async function readConsumtionData(): Promise<void> {
    for (let i = 0; i < devices.length; i++) {
        //get d-raw; w-raw; m-raw
        let dev: any = devices[i].split('.');
        let state = getState(devices[i]).val;
        state = state.slice(1,-1);
        state = state.replaceAll('null', '0');
        await writeConsumtionData(devices[i].slice(0,-5), dev[dev.length -1], Array.from(state.split(','), Number));
    }
}
readConsumtionData();

on({ id: [].concat(Array.prototype.slice.apply($('daikin-cloud.*.lastUpdateReceived'))), change: 'any' }, async (obj) => {
    await readConsumtionData();
});
