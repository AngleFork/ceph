### Context
```C++
class GenContext{
    virtual void finish(T t);
    void complete(C &&t){
        finish(std::forward<C>(t));
        delete this;
    }
}
```

### OSDOp
```C++
struct OSDOp{
    ceph_osd_op op; //Each individual object operation. 
    sobject_t soid;
    bufferlist indata, outdata;
    int32_t rval; //Return value
}
```

### Objecter
```C++
ObjectOperation{
    vector<OSDOp> ops;
    vector<bufferlist*> out_bl;
    vector<Context*> out_handler;
    vector<int*> out_rval;
    OSDOp& add_op(int op){
        add op to ops;
        set placeholder in out_bl, out_handler, out_rval for op;
    }
    void add_data(int op, uint64_t off, uint64_t len, bufferlist& bl){
        add op to ops;
        add bl to indata;
    }
}
class Objector{
    Messager *messager; // Network interface.
    MonClient *monc;
    Finisher *finisher;
    OSDMap *osdmap;
    std::multimap<string, string> crush_location;

    void submit_command(CommandOp *c, ceph_tid_t *ptid);
    void handle_command_reply(McommandReply *m);
    void init();
    void start(const OSDMap *o = nullptr);
    void shutdown();
}
```