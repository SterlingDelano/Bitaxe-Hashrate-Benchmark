Bitaxe Benchmarking Script


This repository contains a Python script designed to benchmark the Hashrate performance of a Bitaxe mining device. Tested on the Bitaxe Supra, should work with Ultra too. Hex not sure do the issue of Power Cycle after each new setting.


The script tests various combinations of core voltages and frequencies to determine the optimal settings for maximum hashrate while maintaining safe operating temperatures.


Dynamic Configuration: Automatically fetches and uses the current default settings of the Bitaxe device.


Temperature Monitoring: Continuously monitors the device temperature during benchmarking to prevent overheating.


Automated Benchmarking: Tests different combinations of core voltage and frequency and records the average hashrate and temperature.


Result Storage: Saves the benchmarking results in a JSON file for future analysis.


Graceful Interruption Handling: Captures interruptions (e.g., Ctrl+C) and resets the device to the best or default settings before exiting.


Cooling Down: Automatically cools down the device between benchmarks if necessary.




Configuration


Before running the script, you can customize the following settings in the script:


    bitaxe_ip: IP address of your Bitaxe device (default: "http://192.168.2.117").


    core_voltages: List of core voltages (in mV) to test (default: [1150, 1200, 1250]).


    frequencies: List of frequencies (in MHz) to test (default: [550, 575, 600]).


    cool_down_voltage: Voltage to use during cooldown (default: 1166 mV).


    cool_down_frequency: Frequency to use during cooldown (default: 400 MHz).


    cool_down_time: Duration of cooldown between benchmarks (in seconds, default: 300 seconds).


    benchmark_time: Duration of each benchmark (in seconds, default: 9000 seconds).


    sample_interval: Interval between fetching system data during benchmarking (in seconds, default: 150 seconds).


    max_temp: Maximum allowed temperature before stopping a benchmark (in °C, default: 66°C).


    max_allowed_voltage: Maximum allowed core voltage (in mV, default: 1300 mV). DO ONLY MODIFY IF YOU KNOW WHAT YOU ARE DOING!


    max_allowed_frequency: Maximum allowed frequency (in MHz, default: 600 MHz). DO ONLY MODIFY IF YOU KNOW WHAT YOU ARE DOING!




Usage


To run the script:

    python bitaxe_hashrate_benchmark.py



The script will perform the following actions:


Fetch the default core voltage and frequency from the Bitaxe device.
  
Iterate through the specified voltages and frequencies, applying them to the device and benchmarking their performance.
  
Monitor and record the hashrate and temperature during each benchmark.
  
Save the results to bitaxe_benchmark_results.json.
  
Identify and apply the best performing settings after all benchmarks are completed.

  

Interrupt Handling


If you need to stop the script during benchmarking, simply press Ctrl+C. The script will safely reset the Bitaxe device to the best or default settings and save all results before exiting.
Benchmark Results

After the script finishes, the top 5 performing settings will be displayed in the terminal and saved in the results file.
Example Output:


    Top 5 Performing Settings:

    Rank 1:
    Core Voltage: 1200mV
    Frequency: 590MHz
    Average Hashrate: 555.5 GH/s
    Average Temperature: 60°C
    Efficiency: 23.01 J/TH

    Rank 2:
    Core Voltage: 1250mV
    Frequency: 575MHz
    Average Hashrate: 666.6 GH/s
    Average Temperature: 62°C
    Efficiency: 22.34 J/TH
Contributions are welcome! Please feel free to submit a pull request or open an issue to suggest improvements.
Acknowledgments


Special thanks to the Bitaxe community and OSMU Discord for their continued support and feedback.
Thank you for sharing your Python script designed to benchmark the Hashrate performance of a Bitaxe mining device! I’ve reviewed its features, configuration options, and overall functionality, and I’d like to suggest several improvements that could enhance its usability, flexibility, and robustness. These suggestions are based on the script’s current design and the potential needs of users working with Bitaxe Supra, Ultra, and possibly Hex devices. Below, I’ll outline my proposed enhancements, explain their benefits, and suggest how they could be implemented.
Proposed Improvements
1. Add Command-Line Arguments for Configuration
Why?
Currently, users must edit the script to adjust settings like bitaxe_ip, core_voltages, frequencies, and benchmark_time. This can be inconvenient and error-prone. Adding command-line arguments would allow users to customize these parameters without modifying the code, improving accessibility and ease of use.
How?
Use Python’s argparse module to define configurable options. For example:
python
import argparse

parser = argparse.ArgumentParser(description='Benchmark Bitaxe mining device.')
parser.add_argument('--ip', default='http://192.168.2.117', help='IP address of the Bitaxe device')
parser.add_argument('--voltages', nargs='+', type=int, default=[1150, 1200, 1250], help='List of core voltages to test (mV)')
parser.add_argument('--frequencies', nargs='+', type=int, default=[550, 575, 600], help='List of frequencies to test (MHz)')
parser.add_argument('--benchmark-time', type=int, default=9000, help='Duration of each benchmark (seconds)')
# Add similar arguments for cool_down_voltage, max_temp, etc.
args = parser.parse_args()
Then, replace hardcoded variables with args.ip, args.voltages, etc. Users could run the script like this:
bash
python bitaxe_hashrate_benchmark.py --ip http://192.168.1.100 --voltages 1100 1150 1200 --frequencies 550 575 --benchmark-time 3600
Benefit:
This makes the script more user-friendly and flexible, especially for users testing multiple devices or settings without needing to edit the source code.
2. Implement Progress Indicators and Real-Time Monitoring Output
Why?
Each benchmark takes 9000 seconds (2.5 hours) by default, and with 9 combinations (3 voltages × 3 frequencies), the total runtime is over 22 hours. Users have no feedback on progress or current performance during this time, which can feel opaque. Adding progress indicators and real-time data output would improve the experience.
How?  
Progress Indicator: Before each benchmark, display the current combination and total count.
Real-Time Monitoring: During benchmarking, print the hashrate and temperature at each sample_interval (default 150 seconds).
Example implementation:
python
total_combinations = len(args.voltages) * len(args.frequencies)
current_combination = 0

for voltage in args.voltages:
    for frequency in args.frequencies:
        current_combination += 1
        print(f"Testing combination {current_combination}/{total_combinations}: Voltage={voltage}mV, Frequency={frequency}MHz")
        # Set voltage and frequency here
        for _ in range(int(args.benchmark_time / args.sample_interval)):
            # Fetch and record data
            hashrate = fetch_hashrate()  # Placeholder for actual function
            temp = fetch_temperature()   # Placeholder for actual function
            print(f"Current Hashrate: {hashrate} GH/s, Temperature: {temp}°C")
            time.sleep(args.sample_interval)
Benefit:
Users can track progress and monitor device performance in real-time, making long benchmarking sessions more transparent and engaging.
3. Automatically Timestamp the Results File
Why?
The script saves results to bitaxe_benchmark_results.json, overwriting previous runs. This risks losing data if users forget to rename the file manually. Adding a timestamp to the filename ensures each run’s results are preserved.
How?
Generate a timestamped filename using the current date and time:
python
from datetime import datetime

timestamp = datetime.now().strftime("%Y-%m-%d_%H-%M-%S")
results_file = f"bitaxe_benchmark_results_{timestamp}.json"
# Use results_file when saving results
Example output filename: bitaxe_benchmark_results_2023-10-15_14-30-00.json.
Benefit:
Prevents accidental data loss and allows users to keep a history of benchmarking runs without manual intervention.
4. Add a Device Accessibility Check at Startup
Why?
The script assumes the Bitaxe device is reachable at the specified IP address. If it’s not, users might waste time waiting for benchmarks to fail. A preliminary check ensures the device is accessible before starting the long process.
How?
Test connectivity by fetching the default settings (or a simple status endpoint) at the start:
python
import requests
import sys

def check_device_accessibility(ip):
    try:
        response = requests.get(f"{ip}/status", timeout=5)  # Adjust endpoint as needed
        response.raise_for_status()
        print(f"Device reachable at {ip}")
    except requests.RequestException as e:
        print(f"Error: Cannot connect to device at {ip} - {e}")
        sys.exit(1)

check_device_accessibility(args.ip)
Benefit:
Saves time and avoids frustration by catching connectivity issues early.
5. Add Support for Power Cycling Delays (e.g., for Hex Devices)
Why?
You noted uncertainty about the script’s compatibility with the Bitaxe Hex due to “the issue of Power Cycle after each new setting.” If Hex devices require a power cycle or stabilization period after changing voltage/frequency, the script should accommodate this. Without specific details, a configurable delay is a flexible solution.
How?
Add a command-line option for a delay after setting changes:
python
parser.add_argument('--power-cycle-delay', type=int, default=0, help='Delay in seconds after changing settings (e.g., for power cycling)')
After applying new settings:
python
# Set voltage and frequency here
if args.power_cycle_delay > 0:
    print(f"Waiting {args.power_cycle_delay} seconds for device stabilization...")
    time.sleep(args.power_cycle_delay)
Users can set this based on their device’s needs, e.g., --power-cycle-delay 10 for a 10-second wait.
Benefit:
Makes the script adaptable to Hex devices (and others) that may need time to stabilize after settings changes, pending further clarification on Hex behavior.
6. Generate a Human-Readable Summary Report
Why?
While JSON is great for machine processing, a human-readable summary (e.g., a text file) would make results easier to review without external tools, especially for users less comfortable with JSON.
How?
After benchmarking, write a summary to a text file alongside the JSON:
python
with open(f"bitaxe_benchmark_summary_{timestamp}.txt", 'w') as f:
    f.write("Bitaxe Benchmark Summary\n")
    f.write("=======================\n\n")
    for result in results:  # Assuming results is a list of benchmark data
        f.write(f"Voltage: {result['voltage']}mV, Frequency: {result['frequency']}MHz\n")
        f.write(f"Avg Hashrate: {result['avg_hashrate']} GH/s\n")
        f.write(f"Avg Temperature: {result['avg_temperature']}°C\n")
        f.write(f"Efficiency: {result['efficiency']} J/TH\n\n")
Benefit:
Provides an accessible, at-a-glance view of all tested settings, complementing the JSON output.
Additional Considerations
While the above are my top recommendations, here are a few more ideas that could be explored:
Support Voltage/Frequency Ranges: Allow users to specify ranges with steps (e.g., --voltages-range 1100 1300 50 for 1100 to 1300 mV in 50 mV steps) using range(start, stop, step). This increases flexibility but might need UI refinement.
Resume Interrupted Benchmarks: Save intermediate results after each benchmark and add a --resume flag to continue from the last completed test. Useful for long runs but adds complexity.
Enhanced Statistics: Calculate and display variance or stability metrics for hashrate/temperature to assess setting reliability. This could refine the “best settings” selection but might be overkill for most users.
To contribute these improvements:
Open an Issue: I could submit an issue on your repository detailing these suggestions, their benefits, and the proposed code snippets for discussion.
Submit a Pull Request: Alternatively, I could fork the repository, implement these changes (starting with the top 6), and submit a pull request with tested code.
For the Hex device issue, I’d recommend adding a note in the README or script comments about potential power cycling needs, alongside the --power-cycle-delay option, until more specifics are provided.              
Thank you for sharing this well-designed tool, and I look forward to seeing how it evolves with community input.
