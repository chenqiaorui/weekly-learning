#### TCP建立连接到断开连接状态变更过程

```
---------------------------------------
客户端              服务端
---------------------------------------
初始状态：CLOSE             LISTEN
---------------------------------------  
三次握手：SYN-SENT          SYN_RCVD       
        
        SYN->
                <-SYN + ACK
        ACK->
---------------------------------------
ESTABLISHED                 ESTABLISHED
        
        request ->
                <- Response
---------------------------------------
四次握手：FIN-WAIT-1         CLOSE_WAIT
        
        FIN->
                <- ACK

FIN-WAIT-2                  LAST_ACK

                <- FIN
        ACK-> 
---------------------------------------
TIME_WAIT and to CLOSE      CLOSE
---------------------------------------

More detail check: https://blog.csdn.net/sulijin/article/details/124412379
```