---
title: Code Tracing - Snort3 Packet Decoding I
date: 2024-09-01
categories: [Network, Network Security]
tags: [NIDS, GDB]
img_path: /assets/img/
---

# Tracing Snort3 with GDB -- Packet Manager and Decode
<details>
<summary>⚠️ Notice: AI-assisted content</summary>

Some sections of this article were generated or assisted by an AI and subsequently edited by the author. While scrutinized, I feel like I still need to manifest a self-declaration: Please report errors or inconsistencies if any. Thanks for your coporation.
</details>

## Purpose of the Article
This article traces a network packet’s journey from Event Binding to the point it can be inspected by Snort Detection Engine. It explains the components and interactions involved — capture, buffering/queueing, the packet manager, and the decoder — and shows how to follow each step with GDB, with an emphasis on the packet manager and the decode pipeline.

## Architecture Intro

Studying Snort’s development history, it's quite fascinating to realize how predescessor developer designed the architecture and make packet processing elegant and efficient.
Snort’s classic packet-processing pipeline is modular and essentially linear: packets are captured from the wire, decoded into protocol-aware data structures, optionally normalized or enriched by preprocessors, and finally evaluated by the detection engine. This high-level flow captures the primary responsibilities and handoffs between components.

![Snort Data Flow](/assets/img/diagram/snort_dataflow.png)

- Packet Capture — acquires raw frames from the network (libpcap, AF_PACKET, etc.) and delivers them into Snort’s internal buffering/queuing system.
- Packet Decoder — parses link, network, transport, and application protocols to produce an annotated packet representation the engine can understand.
- Preprocessors — perform normalization, session reassembly, protocol-specific fixes, and additional metadata enrichment before detection.
- Detection Engine — applies rules and signatures to the decoded/normalized packet and generates alerts or events when matches occur.

This simplified view is useful for our later tracing execution with GDB. While it was adopted in very origin Snort(Sourcefire version), the core concepts and components remain relevant till today's Snort 3, albeit with architectural enhancements and modularity improvements. Following embedded antique slide deck illustrates the Sourcefire-era architecture for reference.

{% raw %}

<div style="text-align: center">
<iframe 
  src="https://www.slideshare.net/slideshow/embed_code/key/v9FRilEXhCp7sI"
  width="510"
  height="420"
  frameborder="0"
  marginwidth="0"
  marginheight="0"
  scrolling="no"
  style="border: var(--border-1) solid #CCC; border-width:1px; margin-bottom:5px; max-width:100%;"
  allowfullscreen>
</iframe>

<div style="margin-bottom:5px">
<strong>
  <a href="https://www.slideshare.net/slideshow/snortppt/265281399" target="_blank">
    snortppt
  </a>
</strong>
from 
<strong>
  <a href="https://www.slideshare.net/SendhilKumar6" target="_blank">Senthil Vit</a>
</strong>
</div>

</div>


{% endraw %}

Graphicalize the architecture into a component diagram, we have the following representation:

![Architecture for Packet Processing](/assets/img/diagram/snort_PacketFlow_component.png) 

You can see the main components involved in packet processing: DAQInstance, Analyzer, Packet Manager, InspectorManager, Detection Engine, etc. Previously we have explored DAQInstance and Event_Bus in detail. In this article, I would like to boostrap the exploration by ask what does snort actually do to understand the packet? How does it know it's a TCP, or UDP packet? Where does the packet come from, and where does it destinate to? All these information are crucial for detection engine to make decision.

### Decoding Component Interaction


![Analyzer to PacketManager](/assets/img/diagram/Analyzer2PacketManager_UML.png)



## Showtime: GDB Tracing

### Breaking Point


### Pipeline and Decode Path


### 