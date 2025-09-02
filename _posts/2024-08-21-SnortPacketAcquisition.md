---
title: Code Tracing - Snort3 Packet Acquisition I
date: 2024-08-20
categories: [Network, Network Security]
tags: [NIDS, GDB]
img_path: /assets/img/
---

# Tracing Snort3 with GDB -- DAQ(Data Acquisition)

## Purpose of the Article

To know how snort acquire packet by the new "Data Acquisition" mechanism, and take a step further to understand pros and cons of it.

https://www.codeproject.com/Articles/685749/Data-Acquisition-Library

[libdaq Github Page README.md](https://github.com/snort3/libdaq)

## Review 

In my [previous blog](../SnortCodeTracing/), we identified two component. The first is `Trough` handling packet stuff in the background, and the second is `Pig`which performs tasks such as analysis and process control in the foreground. Pig feeding their food at Trough. So Trough must be responsible for managing all incoming packets. Therefore, we are now exploring the `packet_io` directory to understand how packets are handled from the moment they enter the physical interface and how Snort acquires these packets and in what form. 

Here is a class definition of `Trough`, and a member variable that worth pay attention to is `pcap_queue`. Because inside 

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

Here is class `SFDAQInstance`.

```
class SFDAQInstance
{
public:
    SFDAQInstance(const char* intf, unsigned id, const SFDAQConfig*);
    ~SFDAQInstance();

    bool init(DAQ_Config_h, const std::string& bpf_string);

    bool start();
    bool was_started() const;
    bool stop();
    void reload();

    DAQ_RecvStatus receive_messages(unsigned max_recv);
    DAQ_Msg_h next_message()
    {
        if (curr_batch_idx < curr_batch_size)
            return daq_msgs[curr_batch_idx++];
        return nullptr;
    }
    int finalize_message(DAQ_Msg_h msg, DAQ_Verdict verdict);
    const char* get_error();

    int get_base_protocol() const;
    uint32_t get_batch_size() const { return batch_size; }
    uint32_t get_pool_available() const { return pool_available; }
    const char* get_input_spec() const;
    const DAQ_Stats_t* get_stats();

    bool can_inject() const;
    bool can_inject_raw() const;
    bool can_replace() const;
    bool can_start_unprivileged() const;
    SO_PUBLIC bool can_whitelist() const;

    int inject(DAQ_Msg_h, int rev, const uint8_t* buf, uint32_t len);
    bool interrupt();

    SO_PUBLIC int ioctl(DAQ_IoctlCmd cmd, void *arg, size_t arglen);
    SO_PUBLIC int modify_flow_opaque(DAQ_Msg_h, uint32_t opaque);
    int set_packet_verdict_reason(DAQ_Msg_h msg, uint8_t verdict_reason);
    int set_packet_trace_data(DAQ_Msg_h, uint8_t* buff, uint32_t buff_len);
    int add_expected(const Packet* ctrlPkt, const SfIp* cliIP, uint16_t cliPort,
            const SfIp* srvIP, uint16_t srvPort, IpProtocol, unsigned timeout_ms,
            unsigned flags);
    bool get_tunnel_bypass(uint16_t proto);

private:
    void get_tunnel_capabilities();

    std::string input_spec;
    uint32_t instance_id;
    DAQ_Instance_h instance = nullptr;
    DAQ_Msg_h* daq_msgs;
    unsigned curr_batch_size = 0;
    unsigned curr_batch_idx = 0;
    uint32_t batch_size;
    uint32_t pool_size = 0;
    uint32_t pool_available = 0;
    int dlt = -1;
    DAQ_Stats_t daq_instance_stats = { };
    uint16_t daq_tunnel_mask = 0;
};
```

## Background Briefing

#### Why uses DAQ?

**What is DAQ? Why uses DAQ? What does it differ from libpcap?** I've been question myself with these question, and a passage from the official document reveal those answers. According to **Snort v2.9.16 User Manual - 1.5 Packet Acquision**]**,`http://manual-snort-org.s3-website-us-east-1.amazonaws.com/node7.html`, DAQ serves as abstracted layer between hardware and software, as same as libpcap,make the network function virtualized and stablized to call. In the context, it explicitly explains: 
> Snort 2.9 introduces the DAQ, or Data Acquisition library, for packet I/O. The DAQ replaces direct calls to libpcap functions with an abstraction layer that facilitates operation on a variety of hardware and software interfaces without requiring changes to Snort. It is possible to select the DAQ type and mode when invoking Snort to perform pcap readback or inline operation, etc.

From LibDAQ gihub [README.md](https://github.com/snort3/libdaq/blob/master/README.md), it mention that same idea with API accoring to different module

> LibDAQ Overview
>> LibDAQ is a pluggable abstraction layer for interacting with a data source (traditionally a network interface or network data plane). Applications using LibDAQ use the library API defined in daq.h to load, configure, and interact with pluggable DAQ modules.

DAQ v.s. libpcap

I'm not saying that we don't need libpcap at all. Instead, default DAQ pcap module relies on the libpcap library for packet capture. Libpcap itself offers different mechanisms to access network packets via lower level kernel interaction, such as using socket-based or memory-mapped function call. While libpcap aim at useing system calls to create sockets, bind interfaces, and capture packets, DAQ modules totally operate in user space, stepping outside of the kernel's protected memory space, which is more secure.

![libdaq call libpacp](/assets/img/network/libdaq_libpacp.png)


### DAQ Element at a Glance

> DAQ Modules
>> The module abstract the low-level details of interacting with various data sources, such as network interfaces, packet capture libraries (like libpcap), or even specialized hardware. And then it modulize whithin multiple parts of API depending on its source type and capabilities. There are two main classes of DAQ modules: base modules and wrapper modules, and these module can be stacked together(forgive me to truncate the details). While starting the Snort, it will setup DAQ module according to DAQ configuration. Module like pcap and AFPacket, or NFQ IPQ IPFW etc. Take pcap for instance, the default module, the module capture raw packets from the network interface or file defined as pcap. Then it preprocess these packets and encapsulate them into DAQ messages for DAQ instance to make further reference. Take AFPacket in the other hand, the DAQ module built on top of the Linux memory-mapped packet socket interface(AF_PACKET). This interface provides direct access to copies of raw packets received on Linux network devices in an adjuct ring buffer. 
>
> DAQ Instances
>> A DAQ instance is an instantiation(`DAQ_Instance_h`) of a DAQ configuration(`DAQ_Config_h`) that contains exactly one base module and zero or more wrapper modules. It define a basic life cycle from initialization, starting, stopping, and destruction. During the instance execution, the DAQ instance retrieves DAQ messages from the underlying DAQ modules so that we can use DAQ library function to inspect the packet protocol and other contents, like `daq_msg_get_pkthdr(msg)` and `daq_msg_get_data(msg)`.
>
> DAQ Messages
>> At its core, LibDAQ is about receiving and processing data. The fundamental unit used for passing data from a DAQ instance up to the application is the DAQ message. Each message is composed of three main components: a type(packet, payload, or flow), a header, and data. And each message is received(`daq_instance_msg_receive`) in vectors from the DAQ instance and finalized(`daq_instance_msg_finalize`) individually.

```
typedef struct _DAQ_Msg {
    DAQ_MsgType type;     // Message type (e.g., DAQ_MSG_TYPE_PACKET)
    uint8_t hdr_len;      // Length of the header in bytes
    void *hdr;            // Pointer to the message header (type-specific)
    uint32_t data_len;   // Length of the data in bytes
    uint8_t *data;         // Pointer to the message data
    void *meta[7];        // Array for additional metadata
    void *priv;            // Pointer for private data 
} DAQ_Msg_h;
```


## Showtime: GDB Tracing

### Where to start

In a previous blog post, I mentioned a helpful tool called Doxygen. Doxygen can work with plugin Graphviz to create visual diagrams of function calls, like the one shown below.

![snort main call graph](/assets/img/diagram/snort_main_callgraph.png)

Let's break down the flow step by step:

1. main: The program starts at the main function.
2. snort_main: From main, the program moves into the `snort_main` function after sets up the traffic mode with the `set_mode` function.
3. main_loop: Finally, `snort_main` sets up and finishes some tasks and then enters the `main_loop` function, which keeps the program running in a loop.

### Dive deeper

We now have a first layer. Then we shall narrow down the topic -- how snort acquire packet. Thus we start by picking some class member function as breakpoints. Also, acknowledging others breakpoint by walking through the code manually is inevitable.

Take snort class for example, GDB can give you a hint if you double click "TAB". 
![gdb break point 2-0](/assets/img/gdb/gdb2-0_setting_breakpoint_snort.png)


After finish breakpoint setting, run the snort. Inside `main` function, `snort` class begin its setup routine.
![gdb break point 2-1](/assets/img/gdb/gdb2-1_snort_setup.png)

While it executes to the **snort.cc:149** and create config object, it triggers that DAQModule to setup(**breakpoint 98**) as well. As we mentioned earlier, DAQ Module serve as low-level interface interacting with data sources(eth0 for instance). According to DAQ Configuration, a specific module(pcap for this case) will be initilize for further use by DAQ Instance.
![gdb break point 2-2](/assets/img/gdb/gdb2-2_daqmodule_setup.png)

Using `print` of gdb to print out what's inside cfg variable, if you're curious.
![gdb break point 2-3](/assets/img/gdb/gdb2-3_daq_loadconfig.png)


Inside the `Pig::prep` function, a DAQ instance is started with the provided data: source, ID, and configuration. This instance is responsible for handling higher-level tasks, such as managing DAQ messages derived from the DAQ module. Therefore the `Pig` class can utilize the `Analyze` function, integrating DAQ Instance with the IPS engine to detect malicious traffic.
![gdb break point 2-4](/assets/img/gdb/gdb2-4_pig_prep.png)


In side `main_loop`, we can observe that there is a while loop, `while ( swine or paused or (Trough::has_next() and !exit_requested) )`, which iteration mainly determining by `Trough::has_next()` function. If we step into `Trough::has_next()`, we can see the returning boolin value is depending on whether a vector, `pcap_queue`, is empty or not -- `return (!pcap_queue.empty() && pcap_queue_iter != pcap_queue.cend());`. Further more, the vector variable declaration, `std::vector<std::string> Trough::pcap_queue;`, indicate that `pcap_queue` is dynamic array(or vector) of string objects(packet data). But how does snort push the packets into the queue??
![gdb break point 2-5](/assets/img/gdb/gdb2-5_trough_hasnext.png)


Now, we can show the hidden thread by `info threads` in gdb. In thread ID 2, there is a `setsockopt()` been called by `ControlMgmt::socket_init()` function when `snort_main` setup stage.
![gdb break point 2-6](/assets/img/gdb/gdb2-6_thread_libpcap.png)

![snort_main call funtion diagram](/assets/img/diagram/main_snortmain_mark1.png)

If you take a further inspectation, inside `socket_init()`, we acknowledge the system call we have been so familiar, such as `socket()`, `bind()` and `listen()` etc. What happen after socket listen and recive a packet?? It's just a begining. Nevertheless, due to the length of the article, I will have to explain the remaining part in the next one. See you at [How does snort handle packet??](../SnortPacketHandle/).
```
int ControlMgmt::socket_init(const SnortConfig* sc)
{
    if (!init_controls())
        FatalError("Failed to initialize controls.\n");

    int sock_family = setup_socket_family(sc);
    if (sock_family == AF_UNSPEC)
        return -1;

    listener = socket(sock_family, SOCK_STREAM, 0);

    if (listener < 0)
        FatalError("Failed to create control listener: %s\n", get_error(errno));

    // FIXIT-M want to disable time wait
    int on = 1;
    setsockopt(listener, SOL_SOCKET, SO_REUSEADDR, &on, sizeof(on));

    if (bind(listener, sock_addr, sock_addr_size) < 0)
        FatalError("Failed to bind control listener: %s\n", get_error(errno));

    if (listen(listener, MAX_CONTROL_FDS) < 0)
        FatalError("Failed to start listening on control listener: %s\n", get_error(errno));

    if (!register_control_fd(listener))
        FatalError("Failed to register listener socket.\n");

    return 0;
}
```