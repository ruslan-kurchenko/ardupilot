# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in the ArduPilot repository.

## Overview

ArduPilot is the most advanced, full-featured, and reliable open source autopilot software available. It's capable of controlling almost any vehicle system imaginable, from conventional airplanes, multi-rotors, and helicopters to rovers, boats, and even submarines.

## Project Structure

### Main Directories

- **ArduCopter/**: Multirotor/helicopter firmware
- **ArduPlane/**: Fixed-wing aircraft firmware
- **Rover/**: Ground/surface vehicle firmware
- **ArduSub/**: Underwater vehicle firmware
- **AntennaTracker/**: Antenna tracking firmware
- **Blimp/**: Airship/blimp firmware
- **libraries/**: Shared libraries used across all vehicles
- **Tools/**: Development tools, tests, and utilities
- **modules/**: Git submodules for external dependencies

### Key Libraries

The `libraries/` directory contains all shared code organized by functionality:
- `AP_HAL*`: Hardware abstraction layer for different boards
- `AP_Motors`: Motor control for various vehicle types
- `AP_GPS`: GPS driver implementations
- `AP_InertialSensor`: IMU/accelerometer/gyroscope drivers
- `AP_Mission`: Mission/waypoint handling
- `AP_Logger`: Data logging functionality
- `AC_AttitudeControl`: Attitude control algorithms
- `AC_WPNav`: Waypoint navigation

## Build System

ArduPilot uses the Waf build system (Python-based). The build system is sophisticated and supports many boards and configurations.

### Basic Build Commands

```bash
# Configure for a specific board
./waf configure --board <board_name>

# Build a specific vehicle
./waf copter      # Build ArduCopter
./waf plane       # Build ArduPlane
./waf rover       # Build Rover
./waf sub         # Build ArduSub

# Common boards
./waf configure --board sitl           # Software-in-the-loop simulator
./waf configure --board Pixhawk1       # Original Pixhawk
./waf configure --board CubeBlack      # Cube Black (Pixhawk 2.1)
./waf configure --board CubeOrange     # Cube Orange
./waf configure --board Pixhawk4       # Pixhawk 4
./waf configure --board MatekF405-Wing # Popular F4 board

# List all available boards
./waf list_boards

# Clean build
./waf clean
./waf distclean  # Full clean including config

# Build and upload
./waf --targets bin/arducopter --upload
```

### Development Commands

```bash
# Run tests
./waf check
./waf check --alltests

# Build with debug symbols
./waf configure --board sitl --debug
./waf copter

# Run specific tests
./waf --targets tests/test_math

# Build examples
./waf examples
```

## Code Style and Standards

### C++ Style Guide

- Use 4 spaces for indentation (no tabs)
- Opening braces on same line for functions and control structures
- Class names use CamelCase (e.g., `AP_GPS`)
- Method names use snake_case (e.g., `get_position()`)
- Constants use UPPER_CASE (e.g., `GPS_MAX_RECEIVERS`)
- Member variables often prefixed with underscore (e.g., `_gps_state`)

### Code Formatting

```bash
# Run astyle code formatter (for astyle-clean directories)
python Tools/scripts/run_astyle.py

# Check Python code style
python Tools/scripts/run_flake8.py

# Run lua checks
./Tools/scripts/run_luacheck.sh
```

### Commit Message Format

- First line: concise summary (50 chars or less)
- Blank line
- Detailed explanation if needed
- Reference issues with #number
- Component prefix (e.g., "Copter: fix altitude hold")

## Testing

### SITL (Software In The Loop)

```bash
# Run SITL simulator
cd Tools/autotest
./sim_vehicle.py -v ArduCopter --console --map

# Different vehicles
./sim_vehicle.py -v ArduPlane
./sim_vehicle.py -v Rover
./sim_vehicle.py -v ArduSub

# With specific options
./sim_vehicle.py -v ArduCopter -f quad -L KSFO --console
```

### Unit Tests

```bash
# Run all unit tests
./waf check

# Build and run specific test
./waf --targets tests/test_math
./build/sitl/tests/test_math

# Run with coverage
python Tools/scripts/run_coverage.py
```

### Autotest

```bash
# Run full test suite for a vehicle
cd Tools/autotest
./autotest.py build.ArduCopter test.ArduCopter

# Run specific test
./autotest.py build.ArduCopter test.ArduCopter.ThrowMode
```

## Common Development Tasks

### Adding a New Parameter

1. Add parameter definition in `Parameters.h`
2. Add to parameter table in `Parameters.cpp`
3. Use `AP_GROUPINFO` macro for proper grouping
4. Document with `@Param` tags for auto-documentation

### Adding a New Mode

1. Create new mode class inheriting from `Mode`
2. Add mode number to defines
3. Register in mode table
4. Implement required virtual methods:
   - `init()`: Mode initialization
   - `run()`: Main loop (typically runs at 400Hz)
   - `name4()`: 4-character mode name

### Working with Libraries

- Libraries should be self-contained
- Use `AP_HAL` for hardware abstraction
- Follow singleton pattern for major subsystems
- Include proper header guards
- Add library to `wscript` if creating new one

### Hardware Driver Development

1. Inherit from appropriate backend class
2. Implement device detection in `detect()` or `probe()`
3. Use `AP_HAL` I2C/SPI interfaces
4. Register driver in board hwdef.dat
5. Add to driver detection list

## Debugging

### GDB with SITL

```bash
# Configure with debug symbols
./waf configure --board sitl --debug

# Build
./waf copter

# Run with GDB
gdb build/sitl/bin/arducopter
(gdb) run --model quad --home -35.363261,149.165230,584,353
```

### Console Output

- Use `hal.console->printf()` for debug output
- `GCS_SEND_TEXT()` for messages to ground station
- Enable debug with `#define DEBUG_ENABLED 1`

### DataFlash Logging

- Define log structure in `LogStructure.h`
- Use `AP::logger().Write()` to log data
- Review logs with MAVExplorer or APM Planner

## Best Practices

### Memory Management

- Avoid dynamic allocation after initialization
- Use static allocation where possible
- Check stack usage with `AP_HAL::Util::available_memory()`
- Be mindful of flash usage on smaller boards

### Performance

- Hot loops run at 400Hz (2.5ms budget)
- Use `AP_HAL::micros()` for timing
- Avoid blocking operations in main thread
- Use scheduler for periodic tasks

### Thread Safety

- Main thread + timer callbacks + optional threads
- Use mutexes/semaphores for shared data
- Avoid priority inversion
- Be careful with ISR contexts

### Error Handling

- Use `AP_HAL::panic()` for fatal errors
- Return error codes rather than silent failures
- Validate parameters and inputs
- Provide meaningful error messages to GCS

## Useful Resources

- Developer Wiki: https://ardupilot.org/dev/
- Discord Developer Chat: https://ardupilot.org/discord
- GitHub Issues: https://github.com/ArduPilot/ardupilot/issues
- Discussion Forum: https://discuss.ardupilot.org/

## CI/CD

The project uses GitHub Actions extensively. Key workflows include:
- Build tests for all supported boards
- SITL testing for each vehicle type
- Unit test execution
- Code style checks
- Coverage analysis

Pull requests automatically trigger relevant CI checks.

## Python Environment Considerations

ArduPilot SITL and tools require Python 3 with specific packages. Common issues and solutions:

### Automated Setup (Recommended)

The `install-prereqs-ubuntu.sh` script now includes NumPy 2.0 compatibility fixes:

```bash
# Run the installation script
./Tools/environment_install/install-prereqs-ubuntu.sh -y

# Activate the venv (for Ubuntu 22.04+)
source ~/venv-ardupilot/bin/activate
```

### SITL Map/Console Not Loading

If `sim_vehicle.py` runs but map/console modules don't load, this is typically due to NumPy 2.0+ compatibility issues:

#### Root Cause: NumPy 2.0+ Compatibility
MAVProxy uses `np.fromstring()` which was removed in NumPy 2.0, causing map module crashes.

#### Solutions:

1. **Use the Fixed Installation Script** (Best):
   ```bash
   ./Tools/environment_install/install-prereqs-ubuntu.sh -y
   source ~/venv-ardupilot/bin/activate
   ```

2. **Manual Fix for Existing Environments**:
   ```bash
   # Downgrade NumPy
   pip install "numpy<2.0" --force-reinstall
   
   # Reinstall MAVProxy to ensure compatibility
   pip install MAVProxy --force-reinstall
   ```

3. **Verify Installation**:
   ```bash
   # Check NumPy version (should be 1.x)
   python -c "import numpy; print(f'NumPy: {numpy.__version__}')"
   
   # Test MAVProxy import
   python -c "import MAVProxy; print('MAVProxy OK')"
   
   # Test GUI libraries
   python -c "import wx, matplotlib; print('GUI libraries OK')"
   ```

#### Environment Mismatch Issues

If using mise/pyenv/asdf, ensure packages are installed in the correct environment:

```bash
# Check which Python is being used
which python3

# Check if packages are in the right place
pip list | grep -E 'MAVProxy|numpy|wxPython'

# If using a different Python manager, create a dedicated venv
python3 -m venv ~/venv-ardupilot
source ~/venv-ardupilot/bin/activate
pip install -r Tools/environment_install/requirements.txt
```

#### System Dependencies

Ensure GTK libraries are installed:
```bash
sudo apt-get install -y libgtk-3-0 libgtk-3-dev libwebkit2gtk-4.1-0 freeglut3-dev
```

### Troubleshooting Steps

1. **Check for NumPy 2.0**:
   ```bash
   python -c "import numpy; print(numpy.__version__)"
   # If output starts with "2.", that's the problem
   ```

2. **Look for the specific error**:
   ```
   cv2.error: OpenCV(4.11.0) :-1: error: (-5:Bad argument) in function 'imdecode'
   > Overload resolution failed:
   >  - buf is not a numpy array, neither a scalar
   ```

3. **Apply the fix**:
   ```bash
   pip install "numpy<2.0" opencv-python==4.10.0.84 --force-reinstall
   ```

### Ubuntu 24.04 Specific Notes

Ubuntu 24.04 is fully supported with the updated installation script. The script automatically:
- Creates a Python virtual environment
- Installs NumPy 1.x for compatibility
- Validates the installation
- Provides warnings if incompatible versions are detected

## Getting Help

When working on ArduPilot code:
1. Search existing issues and PRs first
2. Check developer wiki for guidance
3. Ask on Discord #development channel
4. Post on discuss.ardupilot.org for complex topics
5. Review similar existing code for patterns

Remember: ArduPilot is safety-critical software. Code quality, testing, and reliability are paramount.