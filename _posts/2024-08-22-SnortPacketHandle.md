---
title: Code Tracing - Snort3 Packet Acquisition II
date: 2024-08-20
categories: [Network, Network Security]
tags: [NIDS, GDB]
img_path: /assets/img/
---

# Tracing Snort3 with GDB -- DAQ(Data Acquisition)

## Purpose of the Article

To know how snort acquire packet by the new "Data Acquisition" mechanism, and take a step further to understand pros and cons of it. This article exclusively covers what I have found while go through the GDB debugging process. By writing it down, which others can also easily pick up, it brings more solid comprehension on the topic that I studied. So enjoy!


## Review

In [last blog](../SnortPacketAcquisition/), we have come to understand the basic behavior of the snort main function: **main --> snort_main --> main_loop**.

Let's zoom in to the function which snort spends most of its time of the program life cycle: Inside `main_loop()`, snort repeatly check on whether has next packet comming, happening at the function `Trough::has_next()`. While `Trough::has_next()` is executing, it basically determine whether a vector variable named `pcap_queue` is empty or not and return the boolin value. Here is the breif example of the source code and the illustration.

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
    static std::vector<std::string>::const_iterator pcap_queue_iter;
    static std::string pcap_filter;

    static std::vector<std::string> pcap_queue; <--------+
    static unsigned pcap_loop_count;                     |
    static std::atomic<unsigned> file_count;             |
};                                                       | 
                                                         |
bool Trough::has_next()                                  |
{                                                        |
    return (!pcap_queue.empty() && pcap_queue_iter != pcap_queue.cend());
}
```


## Showtime: GDB Tracing

Let's dive in. First question in my mind when I saw `pcap_queue`, a string vector, on which snort determine whether entering the IPS detecting process or hault while waiting for packet to come. It must manipulated by some sort of network driver IO operation and kernel communication.

Another point of view is that we all know the mechanism of traffic acquisition from [previous lesson](../SnortPacketAcquisition/), then how does it enter next step? According to snort official user manual, it explain that once packet is acquired it will be passed through a series of preprocessing procedure.
![Official User Manual - 5.2 Snort Data Flow](/assets/img/network/NetworkFlow_explain_snort.png)

### How does `pcap_queue` been insert

Since it has been declare inside of `Trough` class, `static std::vector<std::string> pcap_queue;`, we know `pcap_queue` is a vector using <std::string> as template to store the element inside datastructure.



### How does snort cope with those packet

[Last article](../SnortPacketAcquisition/) we disclose there is other thread running `setsockopt()` at the background.

![gdb break point 2-6](/assets/img/gdb/gdb2-6_infothread.png)

As we add more breaking point for the inpection, we realize that once background thread will be brought to the frontend to handle packet. Here comes to the topic we aim at discussing today -- **how does snort cope with those received packet and give it to DAQ Module??**

![gdb break point 2-12](/assets/img/gdb/gdb2-12_packethandle.png)

Interesting thing is that there is a `DataBus::publish()` function before snort handling the funtion, I assert this is some kind of mechanism make snort concurrently proccessing huge amount of connections.
Forgive me for rolling back to where program initialized, setup publish and subscribe function.

![gdb break point 2-1](/assets/img/gdb/gdb2-1_snort_setup.png)

The class `DataBus` acts as a central communication hub for different components of Snort, like `Trough` and `DetectionEngine`. It utilizes a **publish-subscribe** structure, enabling snort to publish events to those subscribers to trigger further reaction.
As you can see in the source code of Binder configuration, for each `PktType`, it determines the corresponding handler and inspector.

![src code binder config](/assets/img/gdb/src_binderconfig_DataBus_SUB.png)

As mentioned previously, the early generation of firewalls, known as stateless firewalls, could only base their decisions on the packet's source address, source port, destination address, destination port, and protocol information. These are often referred to as a five-tuple check. However, a significant drawback of stateless firewalls is that they are completely unaware of previous packets that have already passed through the interface. To overcome this issue, a new generation of firewalls with stateful awareness was invented to distinguish the state of each connection. Nowadays, Snort, as part of firewall components, also needs to meet this requirement.

![gdb break point 2-7](/assets/img/gdb/gdb2-7_binder_configureSUB.png)

Then, after everything is done, I used nmap to scan the host where snort is running on. At the same time, I continue the executing point, till `Trough::has_next()` was hit, and background thread publish the event with the packet information.

![gdb break point 2-8](/assets/img/gdb/gdb2-8_binder_hasnextPUB.png)

Because it was a scanning packet, it will not establish tcp flow state, where firewall tend to recognize it as a "Non Flow Packet" instead of "Flow Session". As the result, it directly calls the function,`NonFlowPacketHandler::handle()`.

![gdb break point 2-9](/assets/img/gdb/gdb2-9_binder_handleEvent.png)

Inside the `NonFlowPacketHandler` class, even a small piece of code that is worth explaining. This code creates a new class named `Binder`, which might cause some confusion. It is important to note that `Binder` actually inherits from the `Inspector` class. What it does is not quite simple: not only does it manage inspection material, it also handles the action of how to process the packet or tcp flow.

![gdb break point 2-10](/assets/img/gdb/gdb2-10_binder_handlepacket.png)

Finally, it retrieve binding information via `get_bindings()` function. Then the notable tipping point is after entering `stuff.apply_action(p)`, it goto the `BA_INSPECT` case.

![gdb break point 2-11](/assets/img/gdb/gdb2-11_binder_PUB.png)

Most of the data acquisition and packet handling procedures conclude here, which is the main focus of this coverage. In the next article, we will discuss how Snort reacts to the packet after `Stuff::apply_action`. See you soon!

![classNonFlowPacketHandler handle call graph](/assets/img/diagram/classNonFlowPacketHandler_callgraph.png)
