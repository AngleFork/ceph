### Thread
Start point:
```C++
void *Thread::_entry_func(void *arg){
    void *r = ((Thread *)arg)->entry_wrapper();
    return r;
}
```

### Acceptor
Only accept new connection, pass it to SimpleMessager.
```C++
SimpleMessager *msgr; // Race
int listen_sd;      // Listen for incomming connections.
int shutdown_rd_fd; // Read shutdown signal from parent thread.
int shutdown_wr_fd; // Write shutdown signal to child thread.

void *Acceptor::entry(){
    ...
    polling for status of listen_sd and shutdown_rd_fd
    if find new connection on listen_sd:
        call msgr->add_accept_pipe(accepted file descriptor)
    if find a shutdown signal on shutdown_rd_fd:
        stop listening and exit.
    ...
}
```

### Pipe
Pipe handles actural reads and writes. 
```C++
class Pipe{
private:
    Reader reader_thread;
    Writer writer_thread;
    DelayedDelivery *delay_thread;

    int sd;
    struct iovec msgvec[SM_IOV_MAX]
public:
    SimpleMessger* msgr;

    char* recv_buffer;
    size_t recv_max_prefetch;
    size_t recv_ofs;
    size_t recv_len;

    Mutex pipe_lock; //Lock this entire pipe
    int state;
    enum{
        STATE_ACCEPTING,
        STATE_CONNECTING,
        STATE_OPEN,
        STATE_STANDBY,
        STATE_CLOSED,
        STATE_CLOSING,
        STATE_WARIT
    }

    Messager::Policy policy;
protected:
    map<int, list<Message*>>out_q; // Priority queue for outbound msgs
    DispatchQueue* in_q;
    list<Message*>sent;
    Cond cond;
}
class Reader{
    Pipe* pipe;
    void *entry(){
        pipe->reader();
        return 0;
    }
}
class Writer{
    Pipe* pipe;
    void *entry(){
        pipe->writer();
        return 0;
    }
}
class DelayedDelivery{
    Pipe *pipe;
    std::deque<pair<utime_t, Message*>>delay_queue;
    Mutex delay_lock;
    Cond delay_cond;
}
```



