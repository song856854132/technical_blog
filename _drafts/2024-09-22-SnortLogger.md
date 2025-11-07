---
title: Code Tracing - Snort3 Packet Acquisition
date: 2024-08-20
categories: [Network, Network Security]
tags: [NIDS, GDB]
img_path: /assets/img/
---

# Tracing Snort3 with GDB 

## Purpose of the Article

## Review

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

```
class Analyzer
{
public:
    enum class State { NEW, INITIALIZED, STARTED, RUNNING, PAUSED, STOPPED, FAILED, NUM_STATES };

    static Analyzer* get_local_analyzer();
    static ContextSwitcher* get_switcher();
    static void set_main_hook(MainHook_f);

    Analyzer() = delete;
    Analyzer(snort::SFDAQInstance*, unsigned id, const char* source, uint64_t msg_cnt = 0);
    ~Analyzer();

    void operator()(Swapper*, uint16_t run_num);

    State get_state() { return state; }
    const char* get_state_string();
    const char* get_source() { return source.c_str(); }

    void set_pause_after_cnt(uint64_t msg_cnt) { pause_after_cnt = msg_cnt; }
    void set_skip_cnt(uint64_t msg_cnt) { skip_cnt = msg_cnt; }

    void execute(snort::AnalyzerCommand*);

    void post_process_packet(snort::Packet*);
    bool process_rebuilt_packet(snort::Packet*, const DAQ_PktHdr_t*, const uint8_t* pkt, uint32_t pktlen);
    void finalize_daq_message(DAQ_Msg_h, DAQ_Verdict);
    void add_to_retry_queue(DAQ_Msg_h, snort::Flow*);

    // Functions called by analyzer commands
    void start();
    void run(bool paused = false);
    void stop();
    void pause();
    void resume(uint64_t msg_cnt);
    void reload_daq();
    void reinit(const snort::SnortConfig*);
    void stop_removed(const snort::SnortConfig*);
    void rotate();
    snort::SFDAQInstance* get_daq_instance() { return daq_instance; }

    bool is_idling() const
    { return idling; }

private:
    void analyze();
    bool handle_command();
    void handle_commands();
    void handle_uncompleted_commands();
    DAQ_RecvStatus process_messages();
    void process_daq_msg(DAQ_Msg_h, bool retry);
    void process_daq_pkt_msg(DAQ_Msg_h, bool retry);
    void post_process_daq_pkt_msg(snort::Packet*);
    void process_retry_queue();
    void set_state(State);
    void idle();
    bool init_privileged();
    void init_unprivileged();
    void term();
    void show_source();
    void add_command_to_uncompleted_queue(snort::AnalyzerCommand*, void*);
    void add_command_to_completed_queue(snort::AnalyzerCommand*);

public:
    std::queue<snort::AnalyzerCommand*> completed_work_queue;
    std::mutex completed_work_queue_mutex;
    std::queue<snort::AnalyzerCommand*> pending_work_queue;

private:
    std::atomic<State> state;
    unsigned id;
    bool exit_requested = false;
    bool idling = false;
    uint64_t exit_after_cnt;
    uint64_t pause_after_cnt = 0;
    uint64_t skip_cnt = 0;
    std::string source;
    snort::SFDAQInstance* daq_instance;
    RetryQueue* retry_queue;
    OopsHandler* oops_handler;
    ContextSwitcher* switcher = nullptr;
    std::mutex pending_work_queue_mutex;
    std::list<UncompletedAnalyzerCommand*> uncompleted_work_queue;
};
```


## Background Briefing

![pig_collaborator diagram](diagram/classPig_collaborate_graph.png)