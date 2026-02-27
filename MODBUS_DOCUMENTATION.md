# Modbus Documentation - Anjoytell Projects

## Overview
This document provides comprehensive Modbus information extracted from all project sources in the Anjoytell workspace, with emphasis on **oi-7010** and **oi-9850** projects.

---

## **Project: OI-7010 (Multi-Channel Gas Monitor)**

### **General Information**
- **Current Version**: 6.1.0 (May 30, 2023)
- **Protocol**: Modbus RTU
- **Role**: Modbus Slave
- **Primary Source Files**:
  - [oi-7010-master/oi-7010-master/files/main.c](anjoytell/oi-7010-master/oi-7010-master/files/main.c)
  - [oi-7010-master/oi-7010-master/files/locals.h](anjoytell/oi-7010-master/oi-7010-master/files/locals.h)
  - [oi-7010-master/oi-7010-master/files/modBus.h](anjoytell/oi-7010-master/oi-7010-master/files/modBus.h)
  - [oi-7010-master/oi-7010-master/files/gdi.c](anjoytell/oi-7010-master/oi-7010-master/files/gdi.c)
  - [oi-7010-master/oi-7010-master/files/datalinker.c](anjoytell/oi-7010-master/oi-7010-master/files/datalinker.c)

### **Communication Settings**
- **Baud Rates Supported**: Configurable (common: 9600, 19200, 38400, 57600, 115200)
- **Data Bits**: 8
- **Stop Bits**: 1
- **Parity**: Configurable (None, Even, Odd)
- **Protocol Support**: 
  - Modbus RTU (default)
  - Otisbus (optional protocol, v5.0.8+)
  - DataCommand (v5.0.8+)
- **Max Protocol Selection**: 2 (DATACOMMAND)

### **Channel Configuration**
The OI-7010 supports multiple channel variants:
- **8 Channel** variant (`CHANNELS_8`)
- **12 Channel** variant (`CHANNELS_12`)
- **32 Channel** variant (`CHANNELS_32`)
- **64 Channel** variant (`CHANNELS_64`)

Each variant includes:
- NUM_MONITORS: 16
- NUM_4_20: 4

### **Key Features**
1. **Multi-Channel Gas Detection**: Supports 8-64 gas sensor channels
2. **Wireless Support**: Laird radio, RFM, Legacy radio (900 MHz & 2.4 GHz)
3. **Primary/Secondary Operation**: Network of monitors with automatic failover
4. **Float Swap Support**: Configurable byte order for float values
5. **Channel Precision**: Configurable decimal precision per channel (v5.2.0+)

### **Modbus Register Map (OI-7010)**

#### **System & Configuration Registers**

| Address | Type | R/W | Description | Data Type | Notes |
|---------|------|-----|-------------|-----------|-------|
| 6000 | Holding | R/W | Modbus Address | uint16 | 1-247 valid |
| 6001-6002 | Holding | R/W | Modbus Baud Rate | uint32 | Low word (6001), High word (6002) |
| 6003 | Holding | R/W | Serial Number Month | uint16 | Build date month |
| 6004 | Holding | R/W | Serial Number Day | uint16 | Build date day |
| 6005 | Holding | R/W | Serial Number Year | uint16 | Build date year |
| 6006 | Holding | R | Serial Number Letter | uint16 | Always 17 ('Q' for OI-7010) |
| 6007-6008 | Holding | R/W | Serial Number | uint32 | Low word (6007), High word (6008) |
| 6009 | Holding | R/W | Float Swap | uint16 | 0=No swap, 1=Swap |
| 6010 | Holding | R/W | Radio Timeout | uint16 | Minutes, 2-255 (min 2 minutes v5.1.8+) |
| 6011 | Holding | R/W | Radio Frequency/Network ID | uint16 | 0-52 for 900MHz, 0-255 for 2.4GHz |
| 6012 | Holding | R | Fault LED Status | uint16 | Current fault LED state |
| 6013 | Holding | R | Monitor Supply Voltage * | float32 | See 6052-6053 (v5.1.2+) |
| 6014 | Holding | R | Monitor Supply Voltage * | float32 | See 6052-6053 (v5.1.2+) |
| 6026-6029 | Holding | R/W | Relay 4 Configuration | varies | Relay settings (fixed v5.0.4) |
| 6034 | Holding | R/W | Calibration Mode | uint16 | 0=Normal, 1=Cal Mode (v5.1.6+) |
| 6035 | Holding | R | 6940 Sensor Detection | uint32 | Bit mask: MSB=Ch1...LSB=Last Ch (v5.1.7+) |
| 6052-6053 | Holding | R | Monitor Supply Voltage (new) | float32 | Low word (6052), High word (6053) (v5.1.2+) |

#### **Channel Data Registers (Per Channel)**
Base address pattern: Channel data starts after system registers

| Offset from Base | Type | R/W | Description | Data Type |
|------------------|------|-----|-------------|-----------|
| +0, +1 | Holding/Input | R | Gas Reading | float32 |
| +2 | Holding/Input | R | Sensor Type | uint16 |
| +3 | Holding/Input | R | Gas Type | uint16 |
| +4, +5 | Holding | R/W | Relay 1 Setpoint | float32 |
| +6, +7 | Holding | R/W | Relay 2 Setpoint | float32 |
| +8, +9 | Holding | R/W | Relay 3 Setpoint | float32 |
| +10, +11 | Holding | R/W | Relay 4 Setpoint | float32 |
| +12, +13 | Holding | R/W | Positive Scale | float32 |
| +14, +15 | Holding | R/W | Negative Scale | float32 |

#### **Channel RSSI Registers (Radio Signal Strength)**
| Address | Type | R/W | Description | Data Type | Version |
|---------|------|-----|-------------|-----------|---------|
| 6201-6233 | Input | R | Channel RSSI (8-ch model) | uint16 | v5.1.5+ |
| 6201-6265 | Input | R | Channel RSSI (32-ch model) | uint16 | v5.1.5+ |
| 6201-6456 | Input | R | Channel RSSI (64-ch model) | uint16 | v5.1.5+ |

#### **Time Since Last Communication Registers**
| Address Pattern | Type | R/W | Description | Notes |
|-----------------|------|-----|-------------|-------|
| Channel-specific | Input | R | Time Since Last Message | -1 = Never received, 0 = Timeout, >0 = Seconds |
| TSLN registers | Input | R | Time Since Last Null | Calibration tracking (v5.1.1+) |
| TLSC registers | Input | R | Time Since Last Cal | Calibration tracking (v5.1.1+) |

#### **Firmware & Diagnostic Registers**
| Address | Type | R/W | Description | Data Type |
|---------|------|-----|-------------|-----------|
| 9900 | Input | R | Revision Major | uint16 |
| 9901 | Input | R | Revision Minor | uint16 |
| 9902 | Input | R | Build Number | uint16 |
| 9981 | Holding | R/W | Reset Flag | uint16 |
| 9982 | Holding | R/W | Soft Reboot | uint16 |

### **Modbus Functions Supported**
- **Function 03**: Read Holding Registers
- **Function 04**: Read Input Registers
- **Function 06**: Write Single Register
- **Function 16**: Write Multiple Registers

### **Protocol Options (v5.0.8+)**
The OI-7010 supports multiple protocols on the same UART:
1. **MODBUS** (0): Standard Modbus RTU
2. **OTISBUS** (1): Proprietary protocol for Otis systems
3. **DATACOMMAND** (2): Data command protocol

### **Fault Codes**
Referenced in code (v5.0.4+):
- **F3**: Bad LPIR (Low Power IR sensor)
- **F5**: NULL_FAULT (Calibration null fault)
- **F6**: CAL_FAULT (Calibration fault)
- **F10**: Various sensor faults
- **F11**: FAULT_TEMP (Temporary fault, doesn't trigger fault relay/terminal)
- **F15**: Radio communication error (sees radio but no valid data)
- **NP**: New Primary (monitor becoming primary)
- **P**: Primary monitor
- **S**: Secondary monitor

### **Recent Major Updates**
- **6.1.0** (05/30/2023): Digi Radio Support
- **6.0.0** (10/03/2022): Different screen support, factory reset changes, 64-channel updates
- **5.2.0** (02/04/2021): RKI variant, channel precision for Modbus, RKI gas types
- **5.1.7** (07/01/2020): Added register 6035 for 6940 sensor detection
- **5.1.6** (03/03/2020): Added register 6034 for calibration mode
- **5.1.5** (01/20/2020): Added channel RSSI registers (6201-6233)

---

## **Project: OI-9850 (Wireless Monitor Controller)**

### **General Information**
- **Current Version**: 3.0.0 (October 6, 2022)
- **Protocol**: Modbus RTU (485 & 232)
- **Role**: Modbus Slave
- **Primary Source Files**:
  - [oi-9850-master/oi-9850-master/9850-1/files/modBus.c](anjoytell/oi-9850-master/oi-9850-master/9850-1/files/modBus.c)
  - [oi-9850-master/oi-9850-master/9850-1/files/modBus.h](anjoytell/oi-9850-master/oi-9850-master/9850-1/files/modBus.h)
  - [oi-9850-master/oi-9850-master/9850-1/files/Registry.c](anjoytell/oi-9850-master/oi-9850-master/9850-1/files/Registry.c)
  - [oi-9850-master/oi-9850-master/9850-1/files/Registry.h](anjoytell/oi-9850-master/oi-9850-master/9850-1/files/Registry.h)
  - [oi-9850-master/oi-9850-master/9850-1/files/local.h](anjoytell/oi-9850-master/oi-9850-master/9850-1/files/local.h)

### **Communication Settings**
- **Interface Options**: RS-485 & RS-232 selectable
- **Baud Rates**: Configurable (9600, 19200, 38400, 57600, 115200)
- **Buffer Sizes**: 
  - TX Buffer: 257 bytes
  - RX Buffer: 257 bytes
- **Protocol**: Modbus RTU
- **Float Swap**: Configurable byte order

### **Channel Configuration**
- **NUM_CHANNELS**: 255 (maximum wireless channels)
- **Supports**: Up to 255 wireless gas sensors

### **Key Features**
1. **Dual Modbus Interface**: RS-485 and RS-232 selectable
2. **Web Interface**: Ethernet-based configuration and monitoring
3. **Primary/Secondary Operation**: Automatic primary failover
4. **Large Channel Count**: Supports 255 wireless sensors
5. **USB Support**: USB Modbus communication (v2.0.1+)
6. **RKI Variant**: Optional RKI branding and features (v2.0.10+)

### **Modbus Register Map (OI-9850)**

#### **System & Configuration Registers**

| Address | Type | R/W | Description | Data Type | Notes |
|---------|------|-----|-------------|-----------|-------|
| 6000 | Holding | R/W | Modbus Address | uint16 | 1-247 valid |
| 6001-6002 | Holding | R/W | Modbus Baud Rate | uint32 | Low word (6001), High word (6002) |
| 6003 | Holding | R/W | Build Month | uint16 | Manufacturing date |
| 6004 | Holding | R/W | Build Day | uint16 | Manufacturing date |
| 6005 | Holding | R/W | Build Year | uint16 | Manufacturing date |
| 6006 | Holding | R | Serial Letter | uint16 | 17 ('Q' letter) |
| 6007-6008 | Holding | R/W | Serial Number | uint32 | Low word (6007), High word (6008) |
| 6009 | Holding | R/W | Float Swap | uint16 | 0=Normal, 1=Swapped, <2 valid |
| 6010 | Holding | R/W | Radio Timeout | uint16 | 0-255 minutes |
| 6011 | Holding | R/W | RF Channel/Network ID | uint16 | Network identifier |
| 6012 | Holding | R | Fault LED Status | uint16 | LED state |
| 6013 | Holding | R | Gen 2 Radio Check | uint16 | 0=Gen1/None, 1=Gen2 (v2.0.8+) |
| 6014 | Holding | R | Radio Frequency Type | uint16 | 0=900MHz/None, 1=2.4GHz (v2.0.8+) |
| 6015 | Holding | R/W | Primary/Secondary | uint16 | 0=Primary, 1=Secondary (v2.0.8+) |
| 6016 | Holding | R | Primary/Secondary Stage | uint16 | 0=Primary, 1-3=Sec/NewPrimary (v2.0.8+) |
| 6017 | Holding | R/W | Factory Default Reset | uint16 | Write 1 to reset (v2.0.8+) |
| 6052-6053 | Holding | R | Monitor Supply Voltage | float32 | Low (6052), High (6053) (v2.0.7+) |

#### **Channel Data Registers (Per Channel)**
Format: Base address + (Channel# * 16)

| Offset | Type | R/W | Description | Data Type |
|--------|------|-----|-------------|-----------|
| +0, +1 | Holding/Input | R | Gas Reading | float32 |
| +2 | Holding/Input | R | Sensor Type | uint16 |
| +3 | Holding/Input | R | Gas Type | uint16 |
| +4, +5 | Holding | R/W | Relay 1 Setpoint | float32 |
| +6, +7 | Holding | R/W | Relay 2 Setpoint | float32 |
| +8, +9 | Holding | R/W | Relay 3 Setpoint | float32 |
| +10, +11 | Holding | R/W | Relay 4 Setpoint | float32 |
| +12, +13 | Holding | R/W | Positive Scale | float32 |
| +14, +15 | Holding | R/W | Negative Scale | float32 |

#### **Channel RSSI Registers (Radio Signal Strength)**
| Address Range | Type | R/W | Description | Version |
|---------------|------|-----|-------------|---------|
| 6201-6456 | Input | R | Radio RSSI per Channel | v2.0.8+ |
| Value = 0 | - | - | Channel unused | - |
| Value > 0 | - | - | RSSI value | - |

#### **Firmware & Diagnostic Registers**
| Address | Type | R/W | Description | Data Type |
|---------|------|-----|-------------|-----------|
| 9275-9277 | Holding | W | Clear Lock Registers | uint16 |
| 9278-9279 | Holding | W | Set Serial Number | uint32 |
| 9280 | Holding | W | Set Month | uint16 |
| 9281 | Holding | W | Set Day | uint16 |
| 9282 | Holding | W | Set Year | uint16 |
| 9899 | Holding | R | Revision Major | uint16 |
| 9900 | Holding | R | Revision Minor | uint16 |
| 9901 | Holding | R | Build Number | uint16 |
| 9981 | Holding | R/W | Reset Flag | uint16 |
| 9982 | Holding | R/W | Modbus Heartbeat | uint16 |
| 9983-9984 | Holding | R | Good Message Count | uint32 |
| 9985-9986 | Holding | R | Bad Message Count | uint32 |
| 9987-9988 | Holding | R | Good Laird Count | uint32 |
| 9989-9990 | Holding | R | Bad Laird Count | uint32 |
| 9991-9992 | Holding | R | Good Legacy Count | uint32 |
| 9993-9994 | Holding | R | Bad Legacy Count | uint32 |
| 9995 | Holding | R | On Time Days | uint16 |
| 9996 | Holding | R | On Time Hours | uint16 |
| 9997 | Holding | R | On Time Minutes | uint16 |
| 9998 | Holding | R | On Time Seconds | uint16 |

### **Modbus Functions Supported**
- **Function 03**: Read Holding Registers
- **Function 04**: Read Input Registers
- **Function 06**: Write Single Register
- **Function 16**: Write Multiple Registers

### **Interface Selection**
The OI-9850 supports two physical Modbus interfaces:
- **MODBUS_485** (0): RS-485 differential
- **MODBUD_232** (1): RS-232 serial

Functions for interface control:
```c
void Enable485(void);      // Enable RS-485 interface
void Enable232(void);      // Enable RS-232 interface
void SetPick485or232(uint8_t pick);  // Select interface
```

### **USB Modbus Support**
The OI-9850 includes USB-based Modbus communication:
```c
uint8_t USBmodBusRun(uint8_t *buffer, uint8_t *head, uint8_t *tail, uint8_t length);
```

### **Fault Codes (v2.0.0+)**
- **F3**: BAD_LPIR (Low Power IR sensor fault)
- **F5**: NULL_FAULT (Null calibration fault)
- **F6**: CAL_FAULT (Calibration fault)
- **F11**: FAULT_TEMP (Temporary fault, no fault output trigger)

### **Recent Major Updates**
- **3.0.0** (10/06/2022): New screen support, fixed 2.4GHz radio network ID display
- **2.0.10** (02/05/2021): RKI variant with web page, FaultFailsafe screen
- **2.0.9** (08/20/2020): Added "6940: PID" display for PID sensors
- **2.0.8** (01/14/2020): Added "volts" gas type, multiple new Modbus registers (6014-6018, 6201-6456, 9900-9902, 9982)
- **2.0.7** (04/11/2019): Added network ID, radio frequency, Pri/Sec to webpage, voltage registers moved to 6052/6053
- **2.0.1** (11/04/2014): Added Modbus validation timer, increased buffer size to 257

---

## **Common Gas Types Supported**
Both systems support extensive gas type libraries:

### **Standard Gases**
- **Combustibles**: LEL (Lower Explosive Limit), CH4 (Methane), %CH4 (Percent Methane)
- **Toxic**: H2S (Hydrogen Sulfide), CO (Carbon Monoxide), CO2 (Carbon Dioxide), SO2 (Sulfur Dioxide)
- **Oxidizer**: O2 (Oxygen)
- **Chlorine Family**: Cl2 (Chlorine), ClO2 (Chlorine Dioxide), HCl (Hydrogen Chloride)
- **Nitrogen Compounds**: NH3 (Ammonia), NO (Nitric Oxide), NO2 (Nitrogen Dioxide), N2O (Nitrous Oxide)

### **Specialty Gases** (v5.1.3+, v2.0.6+)
- **Hydrides**: AsH3 (Arsine - v5.1.2), PH3 (Phosphine - v3.0.3), SiH4 (Silane)
- **Organics**: EtO (Ethylene Oxide - v5.1.1), CH3SH (Methyl Mercaptan - v5.1.1), CH2O (Formaldehyde - v3.0.2)
- **Halogens**: HBr (Hydrogen Bromide - v5.1.0/v2.0.4), HF (Hydrogen Fluoride)
- **Refrigerants** (v5.1.3/v2.0.6): R410A, R1234YF, R32
- **Other**: O3 (Ozone - v3.0.2), VOC (Volatile Organic Compounds)
- **Non-Gas**: INCHES (Tank level), 4-20 (4-20mA sensors), VOLTS (Voltage - v2.0.8)

---

## **Shared Features Across Projects**

### **Primary/Secondary Network Operation**
Both systems support Primary/Secondary monitor configurations:
- **Primary Monitor**: Network coordinator (Address 1)
- **Secondary Monitors**: Backup units
- **Automatic Failover**: Secondary becomes Primary if Primary fails
- **New Primary State**: Transition state during failover

### **Radio Support**
Both support multiple radio types:
- **Laird Radios**: 900 MHz and 2.4 GHz
- **Legacy Radios**: Older 900 MHz systems
- **RFM Radios**: Alternative radio modules
- **Network IDs**: Configurable using Modbus register 6011

### **Float Data Handling**
Both systems support configurable float byte order:
- **Normal Mode** (byFloatSwap = 0): Standard byte order
- **Swapped Mode** (byFloatSwap = 1): Reversed byte order
- Controlled via register 6009

### **4-20mA Sensor Support**
Both systems can interface with 4-20mA sensors with:
- Configurable scaling
- Precision adjustment
- Sensor type recognition

---

## **BOM & Parts Data (repo to ai folder)**

### **CSV Files with Modbus References**
Multiple BOM (Bill of Materials) CSV files contain references to Modbus-related components:

#### **Files with Modbus Content**
- `bom csv fishy/OI-6000K-NXP.csv` - Lines 7290, 8600
- `bom csv fishy/OI-6900-C-2B.csv` - Lines 11168, 11686
- `bom csv fishy/bom parts/Full Parts.2.csv` - Numerous references (1394-4958)
- `bom csv fishy/bom parts/Parts 1301-1400.csv` - Line 94
- `bom csv fishy/bom parts/Parts 1501-1600.csv` - Lines 48, 74
- `bom csv fishy/bom parts/Parts 2001-2100.csv` - Lines 8, 10, 24
- `bom csv fishy/bom parts/Parts 4201-4300.csv` - Lines 36-101 (extensive)
- `bom csv fishy/bom parts/Parts 4301-4400.csv` - Lines 2-30 (RS-485 related)
- `bom csv fishy/bom parts/Parts 4501-4600.csv` - Lines 21, 45-47
- `bom csv fishy/bom parts/Parts 4901-5000.csv` - Lines 53-59

#### **Component Types in BOMs**
Based on filenames, likely Modbus-related components include:
- RS-485 transceivers (IC-RXTX 485)
- MCU/Microcontrollers (PIC series with UART)
- Power regulators
- Connectors
- UARTs and communication ICs

---

## **Integration Notes**

### **Connecting to OI-7010/OI-9850 via Modbus**
1. **Physical Connection**:
   - OI-7010: RS-485 (primary), verify baud rate
   - OI-9850: RS-485 or RS-232 (selectable)
   
2. **Communication Parameters**:
   ```
   Baud Rate: 9600 (default, configurable)
   Data Bits: 8
   Stop Bits: 1
   Parity: None (default, configurable)
   ```

3. **Reading a Gas Value** (Function 03):
   ```
   Address: Slave address (from register 6000)
   Function: 03 (Read Holding Registers)
   Register: Channel base + 0 (for float low word)
   Count: 2 (float32 = 2 registers)
   ```

4. **Setting an Alarm Point** (Function 16):
   ```
   Address: Slave address
   Function: 16 (Write Multiple Registers)
   Register: Channel base + 4 (Relay 1 low word)
   Count: 2 (float32 = 2 registers)
   Data: [Low word, High word] (check float swap setting)
   ```

5. **Checking System Status**:
   - Read Fault LED (6012)
   - Read Primary/Secondary state (6015/6016 for OI-9850)
   - Monitor RSSI values (6201+ for signal strength)

### **Best Practices**
1. **Float Handling**: Always check register 6009 for float swap setting before reading/writing float values
2. **Error Handling**: Check response codes; MS_BADADDR (0x02) indicates invalid register
3. **Channel Iteration**: Channel data is sequential; calculate base address per channel
4. **RSSI Monitoring**: Use RSSI registers (6201+) to monitor wireless link quality
5. **Timeout Settings**: Radio timeout (6010) should be set appropriately for your network (minimum 2 minutes for OI-7010 v5.1.8+)
6. **Primary/Secondary**: Monitor register 6015/6016 to detect failover events

---

## **Troubleshooting Common Modbus Issues**

### **No Response from Device**
- Verify baud rate matches device setting (registers 6001-6002)
- Check Modbus address (register 6000)
- Confirm physical connection (RS-485 A/B polarity, termination)
- OI-9850: Verify correct interface selected (485 or 232)

### **Incorrect Float Values**
- Check float swap setting (register 6009)
- Verify byte order handling in your Modbus master
- Odd register is typically LSW, even register is MSW (when not swapped)

### **Register Access Errors**
- MS_BADADDR (0x02): Invalid register address
- Verify channel exists and is enabled
- Check register is within valid range for your model
  - OI-7010: 8-64 channels (model dependent)
  - OI-9850: Up to 255 channels

### **Communication Dropouts**
- Check RSSI values (registers 6201+) for weak signals
- Verify radio timeout (register 6010) is adequate
- Monitor good/bad message counts (9983-9994 on OI-9850)
- Check for electromagnetic interference

---

## **API Examples**

### **Python (pymodbus) Example**
```python
from pymodbus.client import ModbusSerialClient
import struct

# Connect to OI-7010
client = ModbusSerialClient(
    port='COM3',
    baudrate=9600,
    timeout=1,
    parity='N',
    stopbits=1,
    bytesize=8
)

# Read gas concentration from channel 1
# Assuming float swap = 0, Channel 1 base = 0
def read_channel_gas(channel):
    base_addr = (channel - 1) * 16  # Simplified calculation
    result = client.read_holding_registers(base_addr, 2, slave=1)
    if not result.isError():
        # Combine two registers into float32
        raw_value = struct.pack('>HH', result.registers[0], result.registers[1])
        gas_value = struct.unpack('>f', raw_value)[0]
        return gas_value
    return None

# Read modbus address
result = client.read_holding_registers(6000, 1, slave=1)
if not result.isError():
    modbus_addr = result.registers[0]
    print(f"Device Modbus Address: {modbus_addr}")

# Read firmware version
result = client.read_holding_registers(9900, 3, slave=1)
if not result.isError():
    major = result.registers[0]
    minor = result.registers[1]
    build = result.registers[2]
    print(f"Firmware: v{major}.{minor}.{build}")

client.close()
```

### **C Example (libmodbus)**
```c
#include <modbus.h>
#include <stdio.h>
#include <stdint.h>

int main() {
    modbus_t *ctx;
    uint16_t tab_reg[64];
    float gas_value;
    
    // Create Modbus RTU context
    ctx = modbus_new_rtu("/dev/ttyUSB0", 9600, 'N', 8, 1);
    modbus_set_slave(ctx, 1);
    
    if (modbus_connect(ctx) == -1) {
        fprintf(stderr, "Connection failed\n");
        modbus_free(ctx);
        return -1;
    }
    
    // Read channel 1 gas value (2 registers for float32)
    if (modbus_read_registers(ctx, 0, 2, tab_reg) == 2) {
        // Combine registers into float (assuming no swap)
        uint32_t raw = ((uint32_t)tab_reg[0] << 16) | tab_reg[1];
        memcpy(&gas_value, &raw, sizeof(float));
        printf("Gas Value: %.2f\n", gas_value);
    }
    
    // Read device serial number (2 registers)
    if (modbus_read_registers(ctx, 6007, 2, tab_reg) == 2) {
        uint32_t serial = ((uint32_t)tab_reg[1] << 16) | tab_reg[0];
        printf("Serial Number: %u\n", serial);
    }
    
    modbus_close(ctx);
    modbus_free(ctx);
    return 0;
}
```

---

## **Document Generation Info**
- **Generated**: February 2, 2026
- **Source Workspace**: d:\\anjoytell
- **Primary Projects Analyzed**: 
  - oi-7010-master (v6.1.0)
  - oi-9850-master (v3.0.0)
- **Additional Data**: BOM CSV files in `repo to ai` folder

---

## **Appendix: File Locations**

### **OI-7010 Source Files**
- Main: `anjoytell/oi-7010-master/oi-7010-master/files/main.c`
- Headers: `anjoytell/oi-7010-master/oi-7010-master/files/locals.h`
- Modbus: `anjoytell/oi-7010-master/oi-7010-master/files/modBus.h`
- Modbus Implementation: `anjoytell/oi-7010-master/oi-7010-master/files/modBus.c`
- Register Stubs: `anjoytell/oi-7010-master/oi-7010-master/files/mbStubs.c`

### **OI-9850 Source Files**
- Main: `anjoytell/oi-9850-master/oi-9850-master/9850-1/files/main.c`
- Headers: `anjoytell/oi-9850-master/oi-9850-master/9850-1/files/local.h`
- Modbus: `anjoytell/oi-9850-master/oi-9850-master/9850-1/files/modBus.h`
- Modbus Implementation: `anjoytell/oi-9850-master/oi-9850-master/9850-1/files/modBus.c`
- Registry: `anjoytell/oi-9850-master/oi-9850-master/9850-1/files/Registry.c`
- Modbus 232: `anjoytell/oi-9850-master/oi-9850-master/9850-1/files/Modbus232.c`

---

**End of Document**
