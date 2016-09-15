# Duck Documentation

<!---
	This file looks much better if you open it from here:
		https://github.com/tomirendo/lab_repo/blob/master/Full_Code/readme.MD

	Alternatively, you can open this file locally in Chrome if you have the "MarkDown Preview Plus" plugin with "Allow access to file URLs" enabled (can be done here: "chrome://extensions/")

	To see the Math properly, check "Enable LaTeX delimiters" in the "MarkDown Preview Plus" plugin settings.
	
-->

### DAC-ADC parts and assembly

![](http://i.imgur.com/j1jujAp.jpg=100x20)

The product is based on openDACs's recipe. For more details:
http://opendacs.com/dac-adc-homepage/

### Output Voltage
The minimum step of the output voltage is approx:

$$ \Delta V = \frac{Max - Min}{2^{bits}} = \frac{10 - (-10)}{2^{16}} \approx 0.3\cdot10^{-3} volt $$

with this output voltage we can get a clear signal for $AC \approx 30 \cdot 10 ^{-3}\  volt$, and a good signal for $AC \approx 3 \cdot 10 ^-3\  volt$ (RMS).
### Communication
We copied most of OpenDAC's instructions that you can read about here:
http://opendacs.com/dac-adc-homepage/how-to-communicate-with-the-dac-adc/

The device communicates through the serial port (baudrate 115200) and accepts commands like:

	RAMP1,2,-4,4.8,50,10\r

Ramps the voltage on port 2 from $-4 \ volt$ to $4.8 \ volt$ in 50 steps with 10 microseconds of delay. Commands always end with '\r' (13 in ASCII). The device ignores any '\n's.

### Code structure

The DAC-ADC actions are controlled in the 
    
    void router(std::vector<String> DB)

function. This command accepts a vector of strings (mutable list) called DB that came from the Serial connection. Each string in the vector represents an argument of the given command. for example the command:
    
    RAMP1,2,-4,4.8,50,10\r

Would result in DB[0] being 'RAMP1' and DB[5] being the string '10'. There are Arduino String so DB[5].toInt() would be the number 10.
#### useful functions
    float writeDAC(int dacChannel, float voltage)

Writes the requested voltage to the dacChannel.

    void readADC(byte DB)

Reads the voltage on a single channel (the DB parameter) and writes it though the Serial connection.

    float readADCWithoutPrint(byte DB)

Reads the current voltage of a single channel and return its value as a floating point number. Duck only. 

### The Duck version

The duck version currently has 1 additional command:

* SINE

After the initial connection to the device, the Duck sends a single '\n' that should be cleared from its output (or just read but the software).

## Sine
Command signature:

	SINE,{FREQUENCY},{POINTS_ON_GRAPH},{RAMP_PACE}\r

* FREQUENCY -  The frequency of the AC signal in Hz
* POINTS_ON_GRAPH - The amount of data point the arduino will draw to simulate an AC current. More points would amount to a clearer signal. Too many point would slow the process down and result in a lower frequency.
* RAMP_PACE - Pace of updating the DC voltage on a port measured in Volts per Seconds.

#### Output 
The command outputs:

        Sine with read is running real freq : {real_freq}

Ending with a line break. {real_freq} is replace with the real expected frequency that results from the rounding errors.

#### Example 

    SINE,17,80,0.5\r

Begin SINE state with 17 Hz frequency with 80 points on the graph and ramp pace of 0.5 Volts per second.

### The SINE state
The SINE command differs from other DAC-ADC command in that it doesn't stops. You can only use a small set of commands to update the SINE signal that runs on the selected port. To end the SINE state, you should use the STOPSINE command. the SINE state can be updated with the regular sine command.

There are command during the SINE state, that can affect the reference, AC or DC voltages. 

#### STOPSINE

Signature:

    STOPSINE\r

Stops the SINE mode. All of the Duck ports will produce DC current with the last value (AC+DC) calculated. 

#### AC
Signature :
    
    AC {voltage}:{port}\r

Updates the AC voltage amplitude of the output channel to the {voltage} specified on port number {port}.

Example :
    
    AC 0.03:2\r

Updates the voltage amplitude on port 2 to $30 \ milivolt$.

#### DC

Signature :
    
    DC {voltage}:{port}\r

Updates the DC voltage of port number {port} to the {voltage} specified.

Example :   

    DC 1:3\r

Updates the DC voltage on port number 3 to $1 \ volt$.

### To do
* Stop sine
* Change sine while live
* add instrument Meta data
 * Places to update - CreateInst, SetInstMenu, SetInst, QueryMeta,StartRun

### Extras
* ?Display, Leds?