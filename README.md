--------------------------------------------------------------------------------
Open Power/Performance Analysis Tool (OPPAT)
--------------------------------------------------------------------------------

# Table of Contents
- [Introduction](#introduction)
- [Types of data supported](#oppat-data-supported)
- [OPPAT visualization](#oppat-visualization)
    - [chart features](#chart-features)
    - [chart types](#chart-types)
- [Data collection for OPPAT](#data-collection-for-oppat)
- [Building OPPAT](#building-oppat)
- [Running OPPAT](#running-oppat)
- [Using the browswer GUI Interface](#using-the-browswer-gui-interface)
- [Limitations](#limitations)

--------------------------------------------------------------------------------
## Introduction
Open Power/Performance Analysis Tool (OPPAT) is a cross-OS, cross-architecture Power and Performance Analysis Tool.

The project web page (under construction) is https://patinnc.github.io

For sample visualzation files, download [windows sample html file](sample_html_files/win_mem_bw4.html) or [this Linux sample html file](sample_html_files/lnx_mem_bw4.html) and load them in your browser.

The above files are ~1 second intervals cut from 2 different ~8 second runs. See [the full 8 second Linux run sample html compressed file here](sample_html_files/lnx_mem_bw4_full.html.zip) for a more complete (but compressed) file. You will have to download the file, unzip it, then load it in your browser. The browser URL syntax to load an file is: file:///C:/somepath/lnx_mem_bw4_full.html

--------------------------------------------------------------------------------
## OPPAT data supported
- Linux perf and/or trace-cmd performance files (both binary and text files),
    - perf stat output also accepted
    - So this should work with data from regular Linux or Android
- Windows ETW data (collected by xperf and dumped to text),
- arbitrary power or performance data supported using LUA scripts (so you don't need to recompile the c++ code to import other data (unless the LUA performance becomes an issue))
- read the data files on Linux or Windows regardless of where the files originated (so read perf/trace-cmd files on Windows or ETW text files on Linux)

--------------------------------------------------------------------------------
## OPPAT visualization
See sample_html_files/lnx_mem_bw4.html for an example web file.

OPPAT viz works much better in Chrome (primarily the zoom by touchpad 2 finger scrolling works better on Chrome).

OPPAT has 3 visualization modes:
1. The usual chart mechanism (where OPPAT backend reads the data files and sends data to the browser)
2. You can also create a standalone web page which is the equivalent of the 'regular chart mechanism' but can be exchanged with other users... the standalone web page has the all the scripts and data built-in so it could be emailed to someone and they could load it in their browser. See sample_html_files\lnx_mem_bw4.html, sample_html_files\win_mem_bw4.html and (for a longer version of lnx_mem_bw4) see the compressed file sample_html_files\lnx_mem_bw4_full.html.tgz
3. You can '--save ' a data json file and then --load the file later. The saved json file has is basically the OPPAT data which needs to be sent to the browser by OPPAT.  This avoids re-reading the input perf/xperf files but it won't pick up any changes in charts.json. The full HTML file created with the --web_file option is only slightly bigger than the --save file. This option requires building oppat. See the sample 'saved' files in sample_data_json_files subdir.

Viz general info
- chart all the data in a browser (on Linux or Windows)
- charts are defined in a json file so you can add events and charts without recompiling OPPAT
- browser interface is sort of like Windows WPA (navbar on left)
    - charts are grouped by category (GPU, CPU, Power, etc)
    - charts can be all hidden or selectively displayed
    - hovering over a chart title in the left nav menu scrolls that chart into view
- data from one group of files can be plotted along side a different group
    - so you can say, compare a Linux perf performance vs a Windows ETW run
    - or compare 2 different runs on the same platform
    - a file group tag (file_tag) is prefixed to the title to distinguish charts
    - Charts with the same title are plotted one after the other to allow easy comparison

#### chart features:
- hovering over a section of a line of the chart shows the data point for that line at that point
    - this doesn't work for the vertical lines since they are just connecting 2 points... only the horizontal pieces of each line is searched for the data value
- unlimited zooming in to nanosec level and zooming back out.
    - there are probably orders of magnitude more points to plot than pixels so more data is displayed as you zoom in.
- panning at any level
- a 'small squashed' picture of the full chart is put below each chart with a slider bar so you can navigate around the chart when you are zooming/panning
- charts can be zoomed individually or charts with the same file_tag can be linked so that zooming/panning 1 chart changes the interval of all the charts with the same file_tag
- hovering on a chart legend entry highlights that line.
- clicking on a chart legend entry toggles the visibility of that line.
- double clicking a legend entry makes only that entry visible/hidden
- if a legend entry is hidden and you hover over it, it will be displayed until you hover out
- You can zoom in/out by:
    - mouse wheel vertically on the chart area. The chart zooms on the time in the center of the chart.
         - on my laptop this is doing 2 fingers on vertically on the touchpad
    - clicking on chart and dragging the mouse to the right and releasing the mouse (the chart will zoom to the selected interval)
    - clicking on chart and dragging the mouse to the left and releasing the mouse will zoom out in sort of inversely proportional to how much of the chart you selected. That is, if you left drag almost the whole chart area then the chart will zoom back out ~2x. If you just left drag a small interval then the chart will zoom out the ~whole way.
    - you can also unzoom by (on my laptop) doing a touchpad 2 finger vertical scroll in the opposite direction of zoom 
- You can pan by:
    - on my laptop this is doing 2 fingers on horizontal scroll motion on the touchpad
    - using the slider below the chart 

#### chart types:
- 'cpu busy' chart: a kernelshark-like chart showing the cpu occupancy by pid/thread. See kernelshark reference http://rostedt.homelinux.com/kernelshark/
    - the chart is based on the context switch event and shows the thread running on each cpu at any given time
    - the context switch event is the Linux sched:sched_switch or the Windows ETW CSwitch event.
    - if there are more events than the context switch in the data file then all of the other events are represented as vertical dashes above the cpu.
    - if the events have call stacks then the call stack is also shown in the popup balloon
- line charts
    - The line charts are probably more accurately called step charts since each event (so far) has a duration and these 'durations' are represented by horizontal segments and joined by vertical segments.
    - the vertical part of the step chart can fill a chart if chart lines have a lot of variation
    - you can select (in the left nav bar) to not connect the horizontal segments of each line... so the chart becomes a sort of 'scattered dash' chart. The 'horizonatal dashes' are the actual data points. When you switch from step chart to dash chart, the chart is not redrawn until there is some 'redraw' request (like zoom/pan or highlight (by hovering over a legend entry)).
- stacked charts
    - Stacked charts can cause a lot more data to be generated than line charts. For example, drawing a line chart of when a particular thread is running only depends on that thread. Drawing a stacked chart for running threads is different: a context switch event on any thread will change all the other running threads... so if you have N cpus, you will get N-1 more things to draw per event for stacked charts.
- flamegraphs. For each perf event which has callstacks and is in same file as the sched_switch/CSwitch event, a flamegraph is created.
    - the color of the flamegraph matches the process/pid/tid in the legend of cpu_busy chart... so it is not as pretty as a flamegraph but now the color of a 'flame' actually means something.
    - if you hide a process in the legend (click on the legend entry...it will be greyed out) then the process will not be shown in the flamegraph.
    - if right drag the mouse in the flamegraph that section of the flamegraph will be zoomed 
    - clicking on a 'flame' zooms to just that flame
    - left dragging the mouse in the flamegraph will zoom out
    - clicking a lower level of the flamegraph 'unzooms' to all the data for that level
    - if you click on the 'all' lowest level of the flamegraph you will zoom all the way back out
    - when you click on flamegraph level the chart is resized so that each level is high enough to display the text. This causes the chart to resize. To try and add some sanity to this resizing, I position the last level of the resized chart to bottom of the visible screen.
    - if you go back into the legend and hover over a hidden entry then that entry will be shown in the flamegraph until you hover out
    - if you click on a 'flame' upper level then just that section will be zoomed.
    - if you zoom in/out on the 'parent' chart then the flamegraphs will be redrawn for the selected interval
    - if you pan left/right on the 'parent' chart then the flamegraphs will be redrawn for the selected interval
    - by default the text of each level of the flame graph will probably not fit. If you click on flamegraph then the size will be expanded to enable drawing the text too
    - you can select (in the left nav bar) whether to group the flamegraphs by process/pid/tid or by process/pid or by just process.
    - both 'on cpu', 'off cpu' and 'run queue' flamegraphs are drawn.
        - 'on cpu' is the callstack for what the thread was doing while it was running on the cpu... so the SampledProfile callstacks or perf cpu-clock callstacks indicate what the thread was doing when it was running on the cpu
        - 'off cpu' shows, for threads that not running, how long they were waiting and the callstack when they got swapped out. The swap 'state' (and on ETW the 'reason') is shown as a level above the process. Usually most threads will be sleeping or not running but this helps one to answer the question: "when my thread didn't run... what was it waiting on?". 
            - Showing the 'state' for the context switch lets you see: 
            - for example, whether a thread is waiting on an non-interruptible sleep (state==D on Linux...usually IO)
            - interruptible sleep (state=S... frequently a nanosleep or futex)
        - 'run queue' shows the threads that got swapped and were in a running or runnable state. So this chart shows saturation of the CPU if there are threads in a runnable state but not running.
- I didn't have a canvas-based chart library to use so the charts are kind of crude... I didn't want to spend too much time creating the charts if there is something better out there. The charts have to use the HTML canvas (not SVGs, d3.js etc) due to the amount of data.


--------------------------------------------------------------------------------
## Data collection for OPPAT

Collecting the performance and power data is very 'situational'. One person will want to run scripts, another will want to start measurements with a button, then start a video and then end the collection with the press of a button.
I have a script for Windows and a script for Linux which demonstrates:
- starting data collection, 
- running a workload,
- stop data collection
- post-process the data (creating a text file from the perf/xperf/trace-cmd binary data)
- putting all the data files in an output dir
- creating a file_list.json file in the output dir (which tells oppat the name and type of the output files)

The steps for data collection using the scripts:
- Build spin.exe (spin.x) and wait.exe (wait.x) utilities
    - from the OPPAT root dir do:
    - on Linux: ```./mk_spin.sh```
    - on Windows: ```.\mk_spin.bat``` (from a Visual Studio cmd box)
    - The binaries will be put in the ./bin subdir
- Start with running the provided scripts:
    - run_perf.sh - You need to have trace-cmd and perf installed
        - On Linux, type: ```sudo bash ./scripts/run_perf.sh```
        - By default the script puts the data in dir ../oppat_data/lnx/mem_bw4
    - run_xperf.bat - you need to have xperf.exe installed.
        - On Windows, from a cmd box with admin privileges, type: ```.\scripts\run_xperf.sh```
        - By default the script puts the data in dir ..\oppat_data\win\mem_bw4
    - Edit the run script if you want to change the defaults
    - In addition to the data files, the run script creates a file_list.json file in the output dir. OPPAT uses the file_list.json file to figure out the filenames and the type of files in the output dir.
- The 'workload' for the run script is spin.x (or spin.exe) which does a memory bandwidth test on 1 cpu for 4 seconds and then on all cpus for another 4 seconds.
- Another program wait.x/wait.exe is also started in the background. wait.cpp reads the battery info for my laptop. It works on my dual boot Windows 10/Linux Ubuntu laptop. The sysfs file might have a different name on your Linux and almost surely is different on Android. 
- On Linux, you could probably just generate a prf_trace.data and prf_trace.txt file using the same syntax as is in run_perf.sh but I haven't tried this.
- If you are running on a laptop and want to get the battery power, remember to disconnect the power cable before you run the script.

--------------------------------------------------------------------------------
## Building OPPAT

- On Linux, type ```make``` in the OPPAT root dir
    - If everything works there should be a bin/oppat.x file
- On Windows, you need to:
    - install the Windows version of gnu make. See http://gnuwin32.sourceforge.net/packages/make.htm or, for just the minimum required binaries, use http://gnuwin32.sourceforge.net/downlinks/make.php
    - put this new 'make' binary in the path
    - You need a current Visual Studio 2015 or 2017 c/c++ compiler (I used both the VS 2015 professional and the VS 2017 community compilers)
    - start a Windows Visual Studio x64 native cmd prompt box
    - type ```make``` in the OPPAT root dir
    - If everything works there should be a bin\oppat.exe file
- If you are changing the source code you probably need to rebuild the include dependency file
    - You need to have perl installed
    - on Linux, in the OPPAT root dir do: ```./mk_depends.sh```. This will create a depends_lnx.mk dependency file.
    - on Windows, in the OPPAT root dir do: ```.\mk_depends.bat```. This will create a depends_win.mk dependency file.
- If you are going to run the sample run_perf.sh or run_xperf.bat scripts, then you need to build the spin and wait utilities:
    - On Linux: ```./mk_spin.sh```
    - On Windows: ```.\mk_spin.bat```


--------------------------------------------------------------------------------
## Running OPPAT

- Run the data collection steps [above](#data-collection-for-oppat)
    - now you have data files in a dir (if you ran the default run\_\* scripts:
       - on Windows ..\oppat_data\win\mem_bw4
       - on Linux ../oppat_data/lnx/mem_bw4
    - You need to add the created files to the input_files\input_data_files.json file:
- Starting OPPAT reads all the data files and starts the web server
- on Windows (assuming your data dir is ..\oppat_data\win\mem_bw4)
```
   bin\oppat.exe -r ..\oppat_data\win\mem_bw4 > tmp.txt
```
- on Linux (assuming your data dir is ../oppat_data/lnx/mem_bw4)
```
   bin/oppat.exe -r ../oppat_data/lnx/mem_bw4 > tmp.txt
```
- Now connect your browser to localhost:8081

- You can create a standalone HTML file with the '--web_file some_file.html' option. For example:
```
   bin/oppat.exe -r ../oppat_data/lnx/mem_bw4 --web_file tst2.html > tmp.txt
```
- Then you can load the file into the browser with the URL address: ```file:///C:/some_path/oppat/tst2.html```

--------------------------------------------------------------------------------
## Using the browser GUI Interface
TBD

--------------------------------------------------------------------------------
## Defining events and charts in charts.json
TBD

--------------------------------------------------------------------------------
## Rules for input_data_files.json

- The file 'input_files/input_data_files.json' can be used to maintain a big list of all the data directories you have created.
- You can then select the directory by just specifying the file_tag like:
    - bin/oppat.x -u lnx_mem_bw4 > tmp.txt # assuming there is a file_tag 'lnx_mem_bw4' in the json file.
- The big json file requires you to copy the part of the data dir's file_list.json into input_data_files.json
    - in the file_list.json file you will see lines like:
```json
{"cur_dir":"%root_dir%/oppat_data/win/mem_bw4"},
{"cur_tag":"win_mem_bw4"},
{"txt_file":"etw_trace.txt", "tag":"%cur_tag%", "type":"ETW"},
{"txt_file":"etw_energy2.txt", "wait_file":"wait.txt", "tag":"%cur_tag%", "type":"LUA"}
```
- don't copy the lines like below from the file_list.json file: 
```json
{"file_list":[ 
  ]} 
```
- paste the copied lines into input_data_files.json. Pay attention to where you paste the lines. If you are pasting the lines at the top of input_data_files.json (after the ```{"root_dir":"/data/ppat/"},``` then you need add a ',' after the last pasted line or else JSON will complain.
- for Windows data files add an entry like below to the input_files\input_data_files.json file:
    - yes, use forward slashes:
```json
{"root_dir":"/data/ppat/"},
{"cur_dir":"%root_dir%/oppat_data/win/mem_bw4"},
{"cur_tag":"win_mem_bw4"},
{"txt_file":"etw_trace.txt", "tag":"%cur_tag%", "type":"ETW"},
{"txt_file":"etw_energy2.txt", "wait_file":"wait.txt", "tag":"%cur_tag%", "type":"LUA"}
```

- for Linux data files add an entry like below to the input_files\input_data_files.json file:
```json
{"root_dir":"/data/ppat/"},
{"cur_dir":"%root_dir%/oppat_data/lnx/mem_bw4"},
{"cur_tag":"lnx_mem_bw4"},
{"bin_file":"prf_energy.txt", "txt_file":"prf_energy2.txt", "wait_file":"wait.txt", "tag":"%cur_tag%", "type":"LUA"},
{"bin_file":"tc_trace.dat",  "txt_file":"tc_trace.txt", "tag":"%cur_tag%", "type":"TRACE_CMD"},
{"bin_file":"prf_trace.data", "txt_file":"prf_trace.txt", "tag":"%cur_tag%", "type":"PERF"},

```
[comment]: <> (C:\data\ppat\ppat>make && bin\oppat.exe -u lnx_mem_bwx -r \data\ppat\oppat_data\lnx\mem_bw4 --perf_bin prf_trace.data --perf_txt prf_trace.txt --tc_bin tc_trace.dat --tc_txt tc_trace.txt --lua_txt prf_energy2.txt --lua_bin prf_energy.txt --lua_wait wait.txt > tmp.jnk)

- Unfortunately you have to pay attention to proper JSON syntax (such as trailing ','s
- Here is an explanation of the fields:
    - The 'root_dir' field only needs to entered once in the json file.
        - It can be overridden on the oppat cmd line line with the '-r root_dir_path' option
        - If you use the '-r root_dir_path' option it is as if you had set ```"root_dir":"root_dir_path"``` in the json file
        - the 'root_dir' field has to be on a line by itself.
    - The cur_dir field applies to all the files after the cur_dir line (until the next cur_dir line)
        - the '%root_dir% string in the cur_dir field is replaced with the current value of 'root_dir'.
        - the 'cur_dir' field has to be on a line by itself.
    - the 'cur_tag' field is a text string used to group the files together. The cur_tag field will be used to replace the 'tag' field on each subsequent line.
        - the 'cur_tag' field has to be on a line by itself.
    - For now there are four types of data files indicated by the 'type' field:
         - type:PERF These are Linux perf files. OPPAT currently requires both the binary data file (the bin_file field) created by the ```perf record``` cmd and the ```perf script``` text file (the txt_file field).
         - type:TRACE_CMD These are Linux trace-cmd files. OPPAT currently requires both the binary dat file (the bin_file field) created by the ```trace-cmd record``` cmd and the ```trace-cmd report``` text file (the txt_file field).
         - type:ETW These are Windows ETW xperf data files. OPPAT currently requires only the text file (I can't read the binary file). The txt_file is created with ```xperf ... -a dumper``` command.
         - type:LUA These files are all text files which will be read by the src_lua/test_01.lua script and converted to OPPAT data.
            - the 'prf_energy.txt' file is ```perf stat``` output with Intel RAPL energy data and memory bandwidth data.
            - the 'prf_energy2.txt' file is created by the wait utility and contains battery usage data in the 'perf stat' format.
            - the 'wait.txt' file is created by the wait utility and shows the timestamp when the wait utility began
                - Unfortunately 'perf stat' doesn't report a high resolution timestamp for the 'perf stat' start time

--------------------------------------------------------------------------------
## Limitations
- The data is not reduced on the back-end so every event is sent to the browser... this can be a ton of data and overwhelm the browsers memory
    - I probably should have some data reduction logic but I wanted to get feedback first
    - You can clip the files to a time range: 'oppat.exe -b abs_beg_time -e abs_beg_time'  to reduce the amout of data
        - This is a sort of crude mechanism right now. I just check the timestamp of the sample and discard it if the timestamp is outside the interval. If the sample has a duration it might actually have data for the selected interval...
    - There are many cases where you want to see each event as opposed to averages of events.
    - On my laptop (with 4 CPUs), running for 10 seconds of data collection runs fine.
    - Servers with lots of CPUs or running for a long time will probably blow up OPPAT currently.
    - The stacked chart can cause lots of data to be sent due to how it each event on one line is now stacked on every other line.
- No mechanism yet to put more than 1 event on a chart... say for computing CPI (cycles per instruction).
- The user has to supply or install the data collection software:
    - on Windows xperf
        - See https://docs.microsoft.com/en-us/windows-hardware/get-started/adk-install 
        - You don't need to install the whole ADK... the 'select the parts you want to install' will let you select just the performance tools
    - on Linux perf and/or trace-cmd
        - For perf, try:
```
sudo apt-get install linux-tools-common linux-tools-generic linux-tools-`uname -r`
```

- For trace-cmd, see https://github.com/rostedt/trace-cmd
    - You can do (AFAIK) everything in 'perf' as you can in 'trace-cmd' but I have found trace-cmd has little overhead... perhaps because trace-cmd only supports tracepoints whereas perf supports tracepoints, sampling, callstacks and more.
- Currently for perf and trace-cmd data, you have to give OPPAT both the binary data file and the post-processed text file.
    - Having some of the data come from the binary file speeds things up and is more reliable.
    - But I don't want to the symbol handling and I can't really do the post-processing of the binary data. Near as I can tell you have to be part of the kernel to do the post processing.
- OPPAT requires certain clocks and a certain syntax of 'convert to text' for perf and trace-cmd data.
    - OPPAT requires clock_monotonic so that different file timestamps can be correlated.
    - When converting the binary data to text (trace-cmd report or 'perf script') OPPAT needs the timestamp to be in nanoseconds.
    - see scripts\run_xperf.bat and scripts\run_perf.sh for the required syntax
- given that there might be so many files to read (for example, run_perf.sh generates 7 input files), it is kind of a pain to add these files to the json file input_files\input_data_files.json.
    - the run_xperf.bat and run_perf.sh generate a file_list.json in the output directory.
- perf has so, so many options... I'm sure it is easy to generate some data which will break OPPAT
    - The most obvious way to break OPPAT is to generate too much data (causing browser to run out of memory). I'll probably handle this case better later but for this release (v0.1.0), I just try to not generate too much data.
    - For perf I've tested:
        - sampling hardware events (like cycles, instructions, ref-cycles) and callstacks for same
        - software events (cpu-clock) and callstacks for same
        - tracepoints (sched_switch and a bunch of others) with/without callstacks
- Zooming Using touchpad scroll on Firefox seems to not work as well it works on Chrome


--------------------------------------------------------------------------------
PCM Tools
--------------------------------------------------------------------------------

PCM provides a number of command-line utilities for real-time monitoring:

- pcm : basic processor monitoring utility (instructions per cycle, core frequency (including Intel(r) Turbo Boost Technology), memory and Intel(r) Quick Path Interconnect bandwidth, local and remote memory bandwidth, cache misses, core and CPU package sleep C-state residency, core and CPU package thermal headroom, cache utilization, CPU and memory energy consumption)
- pcm-memory : monitor memory bandwidth (per-channel and per-DRAM DIMM rank)
- pcm-pcie : monitor PCIe bandwidth per-socket
- pcm-iio : monitor PCIe bandwidth per PCIe device
- pcm-numa : monitor local and remote memory accesses
- pcm-power : monitor sleep and energy states of processor, Intel(r) Quick Path Interconnect, DRAM memory, reasons of CPU frequency throttling and other energy-related metrics
- pcm-tsx: monitor performance metrics for Intel(r) Transactional Synchronization Extensions
- pcm-core and pmu-query: query and monitor arbitrary processor core events

Graphical front ends:
- pcm-sensor :  front-end for KDE KSysGuard
- pcm-service :  front-end for Windows perfmon

There is also a utility for reading/writing Intel model specific registers (pcm-msr) supported on Linux, Windows, Mac OS X and FreeBDS.

And finally a daemon that stores core, memory and QPI counters in shared memory that can be be accessed by non-root users.

--------------------------------------------------------------------------------
PCM API documentation
--------------------------------------------------------------------------------

PCM API documentation is embedded in the source code and can be generated into html format from source using Doxygen (www.doxygen.org).

--------------------------------------------------------------------------------
Building the PCM Tools
--------------------------------------------------------------------------------

- Linux: just type 'make'. You will get all the utilities (pcm.x, pcm-memory.x, etc) built in the main PCM directory.
- FreeBSD/DragonFlyBSD: just type 'gmake'. You will get all the utilities (pcm.x, pcm-memory.x, etc) built in the main PCM directory. If the 'gmake' command is not available, you need to install GNU make from ports (for example with 'pkg install gmake').
- Windows: follow the steps in [WINDOWS_HOWTO.rtf](https://raw.githubusercontent.com/opcm/pcm/master/WINDOWS_HOWTO.rtf) (will will need to build or download additional drivers). You can also download PCM binaries from [appveyor build service](https://ci.appveyor.com/project/opcm/pcm/build/artifacts) and required Visual C++ Redistributable from [www.microsoft.com](https://www.microsoft.com/en-us/download/details.aspx?id=48145).
- Mac OS X: follow instructions in [MAC_HOWTO.txt](https://github.com/opcm/pcm/blob/master/MAC_HOWTO.txt)
