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
          bgp table version is 12, local router id is 192.168.2.49
          status codes: (略)
          origin codes: i - IGP, e - EGP, ? - incomplete
          
          Network       Nexthop         Metric  LocPref  Weight  Path
          10.0.0.0      0.0.0.0          0               32768    i
          192.168.2.0   192.168.2.34     0       100        0     i
          192.168.2.0   192.168.2.18     0       100        0     i
          192.168.2.0   192.168.2.50     0       100        0     i

   (8) 檢視 RB 的 BGP 資訊。
   
       
        RB#sh ip bgp
        
          >
          bgp table version is 2, local router id is 192.168.102.102
          status codes: (略)
          origin codes: i - IGP, e - EGP, ? - incomplete
          
          Network       Nexthop         Metric  LocPref  Weight  Path
          10.0.0.0      192.168.2.33     0        100        0     i
                        192.168.2.17     0        100        0     i
          192.168.2.0   192.168.2.66     0        100        0     i
                        192.168.2.33     0        100        0     i
                        192.168.2.17     0        100        0     i 

   (9) 檢視 RC 的 BGP 資訊。
   
        RC#sh ip bgp
        
        
   (10) 查看 RA 的 BGP neighbor 狀態。
   
   
        RA#sh ip bgp neighbors
        
        >
        bgp neighbor is 10.2.2.100, remot AS 65200, external link
        (略)
        bgp version 4, remote router id is 172.16.2.100
        (略)
        --- More ---
        local host: 10.2.2.2(書上可能有錯) or (10.2.2.12) Local Port: 179
        Foreign host: 10.2.2.2.100, Foreign Port: 11003
        (其他細節資訊，略)

   (11) 查看 RA 的 BGP 綜合資訊。
   
         RA#sh ip bgp summary
         >
         bgp router id is 192.168.2.49, local ASN is 65102
         (略)
         Neighbor     Version     AS      MsgRCV  MsgSent       後面欄位不重要，略
         10.2.2.100       4      65200
         192.168.2.18     4      65102
         192.168.2.34     4      65102
         192.168.2.50     4      65102
   
   (12) 清除 RA 的 bgp 資訊。
   
        RA#clear ip bgp *
        RA#sh ip bgp
        
   (13) 查看 RA 路由表。
   
        RA#sh ip route
        
        
   (14) 查看 ISP R1 資訊。
   
        R1#sh run
        
        &&
        
        R1#sh ip route

# BGP Aggregate 路徑集合



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
         AS 65101   - 192.168.1.0 ----------------- ISPR1
                                                   |
                                                   |
                                                   |
                                                   |
                                                   |
                                                   |
                                                 AS 65102
                                                   |
                                                   |
                                                   |
                                               192.168.2.0
                                                   |
                                                   RA
                                                  /  \
                                                 /    \
                                                /      \
                                               /        \
                                              /          \
                                             RC----------RB

* 設定路徑的集合


        RA(config)#router bgp 65102
        RA(config#aggregate address 192.168.2.0 255.255.255.0 summary-only
        // 路徑的集合
        ctrl + Z


* 取消路徑的集合設定

        RA(config)#router bgp 65102
        RA(config#no aggregate address 192.168.2.0 255.255.255.0 summary-only
        // 路徑的集合
        ctrl + Z
        
        
        
 * 檢查
 
 返回步驟 (7)~(11)、(13)、()

 
