
Dispatcher is a internal module of Messager handling received message to the 
upper layer. Messager provides API to register dispatcher and send messages. 
Connection manages the connection status. In fact, we can think Messager as a 
client.
### Dispatcher
```C++
class Dispatcher{
    CephContext* cct;
    bool ms_can_fast_dispatch(const Message*m); // 1. no long-term contended locks
                                                // 2. be able to accept
    bool ms_can_fast_dispatch_any();
    void ms_fast_dispatch(Message* m);

    bool ms_dispatch(Message *m);

    void ms_fast_preprocess(Message *m); // Preprocessing each message.

    void ms_handle_connect(Connection *con); // Callback on newly created connection.
    void ms_handle_fast_connect(Connnectio *con);
    void ms_handle_accept(Connection *con); // Callback on newly accepted connection.
    void ms_handle_fast_accept(Connnection *con);
    void ms_handle_reset(Connection* con); // Callback on newly detected reset.
    void ms_handle_remote_reset(Connection* con);
    void ms_handle_refused(Connection* con); // Callback on refused connection.
}
```
