### Thread
Start point:
```C++
void *Thread::_entry_func(void *arg){
    void *r = ((Thread *)arg)->entry_wrapper();
    return r;
}
```
### Cond
Conditional variable
```C++
pthread_cond_t _c;
Mutex *waiter_mutex;
int Wait(Mutex &mutex); // Release lock and wait for events. After waking up
                        // a lock should be retained.
int Signal(); // Release lock and wake up waiting threads.
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

    void reader(){
        if(state == STATE_ACCEPTING){
            accept();
        }
        while(state != STATE_CLOSED && state != STATE_CONNECTING){
            read message tag
            if(tag == CEPH_MSGR_TAG_KEEPALIVE){
                connection_state->set_last_keepalive(ceph_clock_now())
            }
            if(tag == CEPH_MSGR_TAG_KEEPALIVE2){
                connection_state->set_last_keepalive(ceph_clock_now())
            }
            if(tag == CEPH_MSGR_TAG_MSG){
                Message* m = 0;
                int r = read_message(&m, auth_handler.get());
            }
            if(state == STATE_CLOSED || state == STATE_CONNECTING){
                in_q->dispatch_throttle_release(m->get_dispatch_throttle_size());
                m->put(); //Drop
            }
            if(m->get_seq() <= in_seq){
                in_q->dispatch_throttle_release(m->get_dispatch_throttle_size());
                m->put(); //Drop
            }
            if(m->get_seq() > in_seq + 1){
                missed message.
            }
            m->set_connection(connection_state.get());
            in_q->fast_preprocess(m);
            if(delay_thread){
                inject_delay.
                delay_thread->queue(release, m);
            }else{
                if(in_q->can_fast_dispatch(m)){
                    in_q->fast_dispatch();
                }else{
                    in_q->enqueue(m, m->get_priority(), conn_id);
                }
            }
        }
    }

    void writer(){
        while(state != STATE_CLOSED){
            handle ACK, reconnecting, closing.
            Message* m = _get_next_outgoing();
            if(m){
                sent.push_back(m);
            }
            bufferlist blist = m->get_payload();
            blist.append(m->get_middle());
            blist.append(m->get_data());
            write_message(header, footer, blist);
        }
    }
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
    Mutex delay_lock;// Guard the other variable.
    Cond delay_cond;
    void *entry(){
        while(!stop_delayed_delivery){
            m = delay_queue.pop_front()
            if(pipe->in_q->can_fast_dispatch(m)){
                pipe->in_q->fast_dispatch(m);
            }else{
                pipe->in_q->enqueue(m, m->get_priority(), pipe->conn_id)
            }
        }
    }
}
```

### Messager

### Connection



