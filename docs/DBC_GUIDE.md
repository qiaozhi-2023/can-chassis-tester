# CAN Chassis Tester - DBC Guide

## DBC Files

DBC (Database CAN) files define the structure of CAN messages and signals.

## Building Custom DBC Files

### Basic DBC Structure

```
VERSION ""

NS_ :
    NS_DESC_
    CM_
    BA_DEF_
    BA_
    VAL_
    CAT_DEF_
    CAT_
    FILTER
    BA_DEF_DEF_
    EV_DATA_
    ENVVAR_DATA_
    SGTYPE_
    SGTYPE_VAL_
    BA_DEF_SGTYPE_
    BA_SGTYPE_
    SIG_TYPE_REF_
    VAL_TABLE_
    SIG_GROUP_
    SIG_VALTYPE_
    SIGTYPE_VALTYPE_
    BO_TX_BU_
    BA_DEF_REL_
    BA_REL_
    BA_SGTYPE_REL_
    SG_MUL_VAL_

BS_:

BU_:

BO_ 256 ThrottleCommand: 8 Vector__XXX
 SG_ ThrottlePercent : 0|16@1+ (0.1,0) [0|6553.5] "%" Vector__XXX

BO_ 257 BrakeCommand: 8 Vector__XXX
 SG_ BrakePressure : 0|16@1+ (0.01,0) [0|655.35] "bar" Vector__XXX
 SG_ BrakeForce : 16|16@1+ (1,0) [0|65535] "N" Vector__XXX

BO_ 258 SteeringCommand: 8 Vector__XXX
 SG_ SteeringAngle : 0|16@1- (-0.1,0) [-3276.8|3276.7] "deg" Vector__XXX
 SG_ SteeringTorque : 16|16@1+ (0.01,0) [0|655.35] "Nm" Vector__XXX

BO_ 259 LiftCommand: 8 Vector__XXX
 SG_ LiftHeight : 0|16@1+ (0.01,0) [0|655.35] "mm" Vector__XXX
 SG_ LiftForce : 16|16@1+ (1,0) [0|65535] "N" Vector__XXX
```

## DBC File Components

### Header
- VERSION: DBC format version
- NS_: Namespace declaration

### Nodes
```
BU_: ECU1 ECU2 Gateway
```

### Messages (BO_)
```
BO_ [ID] [MessageName]: [DLC] [Sender]
```
- ID: CAN identifier (decimal)
- MessageName: Human-readable name
- DLC: Data Length Code (0-8 bytes)
- Sender: Node that sends this message

### Signals (SG_)
```
SG_ [SignalName] : [StartBit]|[Length]@[ByteOrder][SignalType] ([Scale],[Offset]) [[Min]|[Max]] "[Unit]" [Receiver]
```
- StartBit: Bit position in message
- Length: Signal length in bits
- ByteOrder: 1=Intel (little-endian), 0=Motorola (big-endian)
- SignalType: + (unsigned), - (signed)
- Scale/Offset: y = x * scale + offset
- Min/Max: Valid range
- Unit: Physical unit
- Receiver: Node receiving this signal

## Example: Throttle Signal

```
BO_ 256 ThrottleCommand: 8 Gateway
 SG_ ThrottlePercent : 0|16@1+ (0.1,0) [0|6553.5] "%" ECU1
```

- Message ID: 256
- Signal: ThrottlePercent
- Start: Bit 0
- Length: 16 bits
- Intel byte order, unsigned
- Scale: 0.1 (raw value × 0.1)
- Offset: 0
- Range: 0 to 6553.5%
- Unit: %

### Value Conversion

```python
# Send 50% throttle:
# Physical value: 50%
# Raw value: 50 / 0.1 = 500

# Receive raw 500:
# Physical value: 500 * 0.1 = 50%
```

## Using DBC Files

### Loading DBC File

```python
from src.dbc.dbc_manager import DBCManager

manager = DBCManager()
db = manager.load("data/dbc_files/chassis_control.dbc")
```

### Working with Signals

```python
# Get signal definition
signal = db.get_message(0x100).get_signal("ThrottlePercent")

# Encode: Convert physical value to bytes
physical_value = 50.0  # 50%
raw_value = signal.physical_to_raw(physical_value)
can_data = codec.encode_signal(physical_value, signal)

# Decode: Convert bytes to physical value
physical_value = codec.decode_signal(can_data, signal)
```

## Creating DBC Files for Your Chassis

### Step 1: Define Messages

What CAN messages does your system need?
- Throttle command
- Brake command
- Steering command
- Status messages

### Step 2: Define Signals

For each message, what signals?
- Signal name
- Data type (8-bit, 16-bit, 32-bit, etc.)
- Scaling factors
- Valid range

### Step 3: Create DBC File

Use a DBC editor or create manually following the format.

### Step 4: Place in `data/dbc_files/`

```
data/dbc_files/
├── chassis_control.dbc
├── brake_system.dbc
└── steering_system.dbc
```

### Step 5: Use in Code

```python
from src.dbc.dbc_manager import DBCManager

manager = DBCManager()
chassis_db = manager.load("data/dbc_files/chassis_control.dbc")
```

## DBC Tools

### Online Editors
- [DBC Editor Online](https://www.kvaser.com/)

### Command Line Tools
- `cantools` Python library
- `dbc-to-json` converters

### Validation

```bash
# Validate DBC file
python -m cantools dump data/dbc_files/chassis_control.dbc
```

## Best Practices

1. **Use meaningful names** - ThrottleCommand not MSG1
2. **Document units** - Always specify units ("%", "deg", "m/s", etc.)
3. **Set valid ranges** - Define min/max for each signal
4. **Use consistent scaling** - 0.1 for % values, 0.01 for high precision
5. **Version your DBC files** - Track changes over time
6. **Test decoding** - Verify scale/offset calculations

## Common Signal Types

| Type | Example | Scale | Offset |
|------|---------|-------|--------|
| Percentage | 0-100% | 0.1 | 0 |
| Angle | -180 to 180° | 0.1 | 0 |
| Force | 0-10000 N | 1 | 0 |
| Pressure | 0-400 bar | 0.01 | 0 |
| Temperature | -40 to 85°C | 0.1 | -40 |
| Speed | 0-200 km/h | 0.1 | 0 |

## Reference

- [DBC Format Specification](https://www.kvaser.com/developer/canlib/html/)
- [cantools Documentation](https://cantools.readthedocs.io/)
- [CAN Message Format](https://en.wikipedia.org/wiki/CAN_bus)
