---
title: Code Tracing - Snort3 main
date: 2024-08-20
categories: [Network, Network Security]
tags: [NIDS, GDB]
img_path: /assets/img/
---
# Tracing Snort3 with GDB
## How to read the source code of a large project?
It's daunting to read exstreamly mass codebase on my own, but I think there might be someone who has been through this before. Therefore I search related keywork to find out what are the most effective way for me to start from a scratch, here are my result:

- Development Documentation
- Start at high level component
- Code tracing

## Purpose of the Article:
By using GDB to trace Snort3, I aim to enhance understanding and facilitate the knowledge of development of larger projects.


## Introduction - What is Snort3?
Snort is an open-source network intrusion prevention system (NIPS) and network intrusion detection system (NIDS). Its history began at 1998, a network security expert, Martin Roesch, develope a lightweight network detection system that response to the existing problems of complexity network environment. Since the firewall at that time simply provide policy which determine acceptance bases on packets' source and destination or protocal, it was so lame in sense of nowaday perspect thus been called static firewall.

At 2001, Martin Roesch found a network security company, Sourcefire Inc, and launch commercial version snort, which can be said to be the predecessor of Firepower, cisco NGFW. 

According to official Cisco security blog, [Snort 3: Rearchitected for Simplicity and Performance](https://blogs.cisco.com/security/snort-3-rearchitected-for-simplicity-and-performance), original snort had no longer compatible and capable to cope with network complexity and speed today. Thus Cisco teams refine snort by provide better multi-pattern search engine and use flow-based module instead of packet-based module.

## Overview of Snort3 Document
Here is the [official document](https://www.snort.org/documents/snort-users-manual-2-9-16-html) of the snort 2.9.16 that I reference from.
### Snort Data Flow
Like wireshark or tcpdump, snort acquire packet from the NIC via libpcap, an open source library used for traffic capture and analysis. Then, packets are passed through a series of decoder to fill out packet structure which determine its Data-Link layer then further decoded for things like Network/Transport layer protocols such as IP, TCP/UDP ports etc.

After decode, packets are sent through the series of preprocessors. Each  preprocessor checks specific offset in this packet, which help trigger following procedure: the detection engine checks each packet against the various options listed in the Snort config files, if positve, enter action mode like alarm or block.

### `dev_notes.txt` in Each Source Code Directory

Take `src/dev_notes.txt` for example， here is the content below. We can obtain the info that there are two components, which under main/* and packet_io/*, corespond to some key feature like services control and packet flow.

> This directory contains the program entry point, thread management, and
control functions.
>* The main / foreground thread services control inputs from signals, the
  command line shell (if enabled), etc.
>* The packet / background threads service one input source apiece.
>
> The main_loop() starts a new Pig when a new source (interface or pcap,
etc.) is available if the number of running Pigs is less than configured.
>
> It also does housekeeping functions like servicing signal flags, shell
commands, etc.
>
> The shell has to be explicitly enabled at build time to be available and
then must be configured at run time to be activated. Multiple simultaneous
remote shells are supported.
>
> Unit test and benchmark testbuild options also impact actual execution.
>
> Reload is implemented by swapping a thread local config pointer by each
running Pig.  The inspector manager is called to empty trash if the main
loop is not otherwise busy.
>
> Reload policy is implemented by cloning the thread local config and 
overwriting the policy map and the inspection policy in the main thread. 
The inspector list from the old config's inspection policy is copied 
into the inspection policy of the new config. After the config pointer 
is cloned, the new inspection policy elements (reloadable) such as 
inspectors, binder, wizard etc are read and instantiated. 
The inspector list of the new config is updated by swapping out the 
old inspectors, binder etc. with the newly instantiated elements. The
reloaded inspectors, binders and other inspection policy elements are 
marked for deletion. After the new inspection policy is loaded, the 
thread local config pointer is swapped with the new cloned config 
by running Pig. This happens in the packet thread. The inspector manager
 is then called to delete any reloaded policy elements and empty trash.

Besides `dev_notes.txt` under src directory, there are dozens of `dev_notes.txt` under others directory such as `packet_io/dev_notes.txt` or `/control/dev_notes.txt` .etc. But I am not goint to rush myself buried with these document so quickly.

```
~$ find ./snot3/src -name dev_notes.txt | wc -l
93
```

## Overview of Snort3 Source Code Architecture
Instead of exam every peice of develope guide, let's take an overview of Snort3 source code structure. From previous discussion of data flow, we know the associate module includes preprocessors, detection engine, and output plugins .etc.

Take a glimps of generic module in Snort3 by displaying directory:

```
$ tree -L 1 ./src
./
├── CMakeLists.txt
├── actions
├── catch
├── codecs
├── connectors
├── control
├── decompress
├── detection
├── dev_notes.txt
├── dump_config
├── events
├── file_api
├── filters
├── flow
├── framework
├── hash
├── helpers
├── host_tracker
├── ips_options
├── js_norm
├── latency
├── log
├── loggers
├── lua
├── lua_wrap.sh
├── main
│   ├── CMakeLists.txt
│   ├── ac_shell_cmd.cc
│   ├── ac_shell_cmd.h
│   ├── analyzer.cc
│   ├── analyzer.h
│   ├── analyzer_command.cc
│   ├── analyzer_command.h
│   ├── bootstrap.lua
│   ├── dev_notes.txt
│   ├── finalize.lua
│   ├── help.cc
│   ├── help.h
│   ├── modules.cc
│   ├── modules.h
│   ├── network_module.cc
│   ├── network_module.h
│   ├── numa.h
│   ├── oops_handler.cc
│   ├── oops_handler.h
│   ├── policy.cc
│   ├── policy.h
│   ├── process.cc
│   ├── process.h
│   ├── reload_tracker.cc
│   ├── reload_tracker.h
│   ├── reload_tuner.h
│   ├── shell.cc
│   ├── shell.h
│   ├── snort.cc
│   ├── snort.h
│   ├── snort_config.cc
│   ├── snort_config.h
│   ├── snort_module.cc
│   ├── snort_module.h
│   ├── snort_types.h
│   ├── swapper.cc
│   ├── swapper.h
│   ├── test
│   ├── thread.cc
│   ├── thread.h
│   ├── thread_config.cc
│   └── thread_config.h
├── main.cc
├── main.h
├── managers
├── memory
├── mime
├── network_inspectors
├── packet_io
├── parser
├── payload_injector
├── policy_selectors
├── ports
├── profiler
├── protocols
├── pub_sub
├── search_engines
├── service_inspectors
├── sfip
├── sfrt
├── side_channel
├── stream
├── target_based
├── time
├── trace
└── utils
```

Whithin such massive code base, where should we dive in? I've been thinking about it since begining of this project, and it ended up impossible for me to solve the conundrum by my own. Thanks to powerful tools, like doxygen, I get to know it better because of call-graph, caller-graph and class inheritation drawn in diagram.

Here is a example of `main` function and its call diagram. Since `main` function is the entry function for most programs, it is definitely worthy to take a look.

![main call funtion diagram](diagram/main_main.png)

Now, let's see the diagram below and compare the source code of `snort_main`, it does two main things. First, initial the socket and the pigs. Second, enter the `main_loop`.

![snort_main call funtion diagram](diagram/main_snortmain.png)

```
static void snort_main()
{
#ifdef SHELL
    ControlMgmt::socket_init(SnortConfig::get_conf());
#endif
    // get configuration
    SnortConfig::get_conf()->thread_config->implement_thread_affinity(STHREAD_TYPE_MAIN, get_instance_id());
    <snip>
    pig_poke = new Ring<unsigned>((max_pigs*max_grunts)+1);
    pigs = new Pig[max_pigs];
    pigs_started = new bool[max_pigs];

    for (unsigned idx = 0; idx < max_pigs; idx++) {
        // initialize pig
    }

    main_loop();

    // clean up pig
    <snip>
#ifdef SHELL
    ControlMgmt::socket_term();
#endif
}
```


Inside `main_loop`, it repeatly handles and processes the packet from src(interface or pcap), and then do service check. From source code, we can see two classes, class `Trough` and class `Pig`, which correspond to packet I/O modules under folder `packet_io` and snort analyzer modules under `main` folder. 

Interestingly, they really do like pigs and absess with it, on which we can find the clue on design methology, they illustrate the concept of snort analyzer interact with packet by analogy with how pigs feed on the trough.

![pig_trough and pig_couch](others/trough_and_couch.jpg)

```
static void main_loop() {
    unsigned max_swine = 0, swine = 0, pending_privileges = 0;

    <snip>
    // Preemptively prep all pigs in live traffic mode
    if (!SnortConfig::get_conf()->read_mode())
        <snip>

    while ( swine or paused or (Trough::has_next() and !exit_requested) ) {
        const char* src;
        int idx = main_read();

        if ( idx >= 0 )
            // handle packet

        if (!pthreads_started)
            // make sure threads of pig start to process

        if ( !exit_requested and (swine < max_pigs) and (src = Trough::get_next()) )
            // packet processing

        service_check();
    }
}
```



From now on, we are going to enter something serious, finally. It'd better for us to scrutinize these two class, `Trough` and `Pig`.

Trough obviously deal with packets from source interface or pcap. For further information, we will have another article to trace data flow from physical interface to trough via DAQ.

```
class Trough
{
public:
    enum SourceType { SOURCE_FILE_LIST, SOURCE_LIST, SOURCE_DIR };

    static void set_loop_count(unsigned c)
    static void set_filter(const char *f);
    static void add_source(SourceType type, const char *list);
    static void setup();
    static bool has_next();
    static const char *get_next();
    static unsigned get_file_count()
    static void clear_file_count()
    static unsigned get_queue_size()
    static unsigned get_loop_count()
    static void cleanup();
private:
    struct PcapReadObject { SourceType type; std::string arg;std::string filter; };

    static bool add_pcaps_dir(const std::string& dirname, const std::string& filter);
    static bool add_pcaps_list_file(const std::string& list_filename, const std::string& filter);
    static bool add_pcaps_list(const std::string& list);
    static bool get_pcaps(const std::vector<struct PcapReadObject> &pol);

    static std::vector<struct PcapReadObject> pcap_object_list;
    static std::vector<std::string> pcap_queue;
    static std::vector<std::string>::const_iterator pcap_queue_iter;
    static std::string pcap_filter;

    static unsigned pcap_loop_count;
    static std::atomic<unsigned> file_count;
};
```

Pig, on the other hand, deal with service control and analyzer. Different from the class `Trough`, `Pig` collaborate with other classes, like class `Analyzer` and class `Swapper`. 

```
class Pig
{
public:
    Pig() = default;
    ~Pig();

    void set_index(unsigned index) { idx = index; }

    bool prep(const char* source);
    void start();
    void stop();

    bool queue_command(AnalyzerCommand*, bool orphan = false);
    void reap_commands();

    Analyzer* analyzer = nullptr;
    bool awaiting_privilege_change = false;
    bool requires_privileged_start = true;

private:
    void reap_command(AnalyzerCommand* ac);
    Swapper* swapper = nullptr;

    std::thread* athread = nullptr;
    unsigned idx = (unsigned)-1;
};
```
![pig_collaborator diagram](diagram/classPig_collaborate_graph.png)
