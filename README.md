# CCNP BGP CLI

* 架構意象圖


                                       AS 65200
                                             \
                                              \
                                               \
                                                \
                                                 \
                                                  \
                                                   \
                   AS 65101   ------------------- ISPR1
                                                   |
                                                   |
                                                   |
                                                   |
                                                   |
                                                   |
                                                 AS 65102
                                                   |
                                                   |
                                                   RA
                                                  /  \
                                                 /    \
                                                /      \
                                               /        \
                                              /          \
                                             RC----------RB

 三個 ASs 彼此均為 BGP 協定的連線，而 AS 65102 內部是以 OSPF 協定連線彼此的。
 
 
 
                                       AS 65200
                                       172.16.1.0
                                       172.16.2.0
                                             \
                                              \
                                               \
                                                \
                                                 \
                                                  \
                                                   \
                   AS 65101   ------------------- ISPR1
                   192.168.1.0                     |
                                               10.2.2.100/24
                                                   |
                                                   |
                                                   |
                                                 AS 65102
                                                   |
                                                   |
                                               10.2.2.12/24
                                                   |
                                                   RA
                                                  /  \ \ 192.168.2.17
                                     192.168.2.49/28  192.168.2.33/28   
                                                 /    \ \
                                                /      \ \
                                               /        192.168.2.34/28
                                 192.168.2.50 /          \ \ 192.168.2.18
                                             RC           RB
                                             |             |
                                  192.168.2.66/28 -------- 192.168.2.65/28
                                  
  # BGP 設定步驟
 
 (1) 首先先將 AS 65102 本身的設定 EIGRP 移除。
 
 https://github.com/QueenieCplusplus/CCNP_EIGRP_2 (EIGRP 設定方式)
 
          RA(config)# no router eigrp 200
          ctrl + Z
          
  (2) 在 RA 的 AS 中，廣播指定的網段。
  
  
          RA(config)#router bgp 65102
          RA(config-router)# network 192.168.2.0 // 此為指定的網段
          ctrl + Z
          
   (3) 檢查 RA 上的設定。
   
          #sh ip bgp
          
          >
          bgp table version is 5, local router id is 192.168.2.49
          status codes: (略)
          origin codes: i - IGP, e - EGP, ? - incomplete
          
          Network       Nexthop    Metric  LocPref  Weight  Path
          192.168.2.0   0.0.0.0     0               32768    i
 
   (4) 設定 IBGP (串在同一 AS 中)
  
        RA(config)#router bgp 65102
        RA(config-router)#neighbor 192.168.2.18 remote-as 65102 // 同上一行的 ASN
        RA(config-router)#neighbor 192.168.2.34 remote-as 65102 
        RA(config-router)#neighbor 192.168.2.50 remote-as 65102 
   
   
   (5) 設定 EBGP (連接相鄰不同 AS 間)
   
        RA(config)#router bgp 65102
        RA(config-router)#network 10.0.0.0
        RA(config-router)#neighbor 10.2.2.100 remote-as 65200
        
        
   (6) 分別檢視 RA、 RB、 RC 的路由設定。
   
        R#sh run
        
        
   (7) 檢視 RA 的 BGP 資訊。
   
       
        RA#sh ip bgp
        
          >
          bgp table version is 5, local router id is 192.168.2.49
          status codes: (略)
          origin codes: i - IGP, e - EGP, ? - incomplete
          
          Network       Nexthop         Metric  LocPref  Weight  Path
          10.0.0.0      0.0.0.0          0               32768    i
          192.168.2.0   192.168.2.34     0                  0     i
          192.168.2.0   192.168.2.18     0                  0     i
          192.168.2.0   192.168.2.50     0                  0     i

        
