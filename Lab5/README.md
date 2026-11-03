## Лабораторная работа №5
### Задание.
1. В офисе С.-Петербург настроить EIGRP.
2. R32 получает только маршрут по умолчанию.
3. R16-17 анонсируют только суммарные префиксы.
4. Использовать EIGRP named-mode для настройки сети.

![Офис Питербург](./Piter.png)               

1. Настраиваем отдельные экземпляры автономных систем:                   
   1.1. autonomous-system 100 для Ipv4.               
   1.2. autonomous-system 200 для Ipv6.
2. По умолчанию все интерфейсы в режим passive -  passive-interface. NO passive-interface включаем только на тех интерфейсах где есть соседи.
3. Командой network прописываем все сети которые хотим анонсировать.
4. Для того чтобы R32 получал только маршруты по умолчанию на интерфейсе e0/3 R16 прописываю команду summary-address.
5. R16-17 анонсируют только суммарные префиксы. Суммарные префиксы будем анонсировать в сторону R18.                           
   5.1 Суммаризация пользовательских сетей 10.128.16.0/ и 10.128.17.0/24 произведем на SW9 на интерфейсах Vlan101 и Vlan102, SW10 на интерфейсах Vlan 103, Vlan 104.                           
   5.2 Суммаризация транзитных сетей выполняется на R16 и R17 интерфейс e0/1, сети 10.0.2.0/30, 10.0.2.4/30, 10.0.2.8/30, 10.0.2.12/30. Команда  summary-address 10.0.2.0 255.255.255.248, summary-address 10.0.2.8 255.255.255.248.                            
6. IPv6.
   6.1 Пользовательские сети fd00::0800/120 и fd00::0900/120. Суммаризируем до fd00::0800/119.
   6.2 Транзитные сети fd00::XXN0/124. XX номер маршрутизатора. N номер интерфейса.

   | Сети транзитные | R16 | R17 | R18 |
   |-------|------|--------|-------|
   | SW9   | fd00::1020/124 | fd00::1100/124 |    |
   | SW10  | fd00::1000/124 | fd00::1120/124 |     |
   | R32   | fd00::1030/124 |         |            |
   | R16   |                |         | fd00::1200/124 |
   | R17   |                |         | fd00::1210/124 |

   сети в сторону R18 суммаризируются summary-address FD00::1000/119                   
   Список Lo
   | Устройства   |     SW9         |       SW10      |       R16       |        R17      |       R18       |         R32     |
   |--------------|-----------------|-----------------|-----------------|-----------------|-----------------|-----------------|
   | Адреса Lo 0  | fd00::ff01:0009 | fd00::ff01:000a | fd00::ff01:0010 | fd00::ff01:0011 | fd00::ff01:0012 | fd00::ff01:0020 |

   сети Lo0 в сторону R18 суммаризируются summary-address FD00::FF01:0/120                      
                     
/ ***************** Настройки роутера R16 ********************** /                     
router eigrp Piter            
 !
 address-family ipv4 unicast autonomous-system 100           
  !          
  af-interface default         
   passive-interface        
  exit-af-interface        
  !       
  af-interface Ethernet0/1           
   summary-address 10.0.2.0 255.255.255.248  / ******* суммаризация транзитных сетей IPv4 в сторону R18  ****** /        
   summary-address 10.0.2.8 255.255.255.248  / ******* суммаризация транзитных сетей IPv4 в сторону R18  ****** /          
   no passive-interface              
  exit-af-interface          
  !            
  af-interface Ethernet0/2                
   no passive-interface               
  exit-af-interface                
  !                
  af-interface Ethernet0/0               
   no passive-interface              
  exit-af-interface             
  !              
  af-interface Ethernet0/3              
   summary-address 0.0.0.0 0.0.0.0 / ********** маршрут по умолчанию в сторону R32 для IPv4 ************** /             
   no passive-interface             
  exit-af-interface            
  !              
  topology base            
  exit-af-topology             
  network 10.0.2.8 0.0.0.3           
  network 10.0.2.12 0.0.0.3           
  network 10.0.2.16 0.0.0.3         
  network 10.0.2.20 0.0.0.3            
  network 10.250.16.250 0.0.0.0            
  eigrp router-id 10.250.16.250            
 exit-address-family                
 !              
 address-family ipv6 unicast autonomous-system 200            
  !               
  af-interface default              
   passive-interface             
  exit-af-interface              
  !           
  af-interface Ethernet0/1             
   summary-address FD00::1000/119 / ******* суммаризация транзитных сетей IPv6 в сторону R18  ****** /            
   summary-address FD00::FF01:0/120   / ******* суммаризация lo IPv6 в сторону R18  ****** /        
   no passive-interface            
  exit-af-interface            
  !           
  af-interface Ethernet0/2            
   no passive-interface             
  exit-af-interface          
  !            
  af-interface Ethernet0/0               
   no passive-interface              
  exit-af-interface             
  !            
  af-interface Ethernet0/3             
   summary-address ::/0    / ********** маршрут по умолчанию в сторону R32 для IPv6 ************** /           
   no passive-interface              
  exit-af-interface              
  !             
  topology base             
  exit-af-topology                  
  eigrp router-id 10.250.16.250             
 exit-address-family          
!                     
/ ****************************** конец настройки R16 ***************************** /              
/ ***************************** диагностические команды R32 ************************** /
Вывод команды show ip route на R32                 
R32#show ip route              
Gateway of last resort is 10.0.2.22 to network 0.0.0.0              
                
D*    0.0.0.0/0 [90/1024640] via 10.0.2.22, 01:05:58, Ethernet0/0            
      10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks         
C        10.0.2.20/30 is directly connected, Ethernet0/0           
L        10.0.2.21/32 is directly connected, Ethernet0/0           
C        10.250.32.250/32 is directly connected, Loopback0                 
Роутер получает только маршрут по умолчанию.            
Вывод команды show ipv6 route на R32               
R32#show ipv6 route                
IPv6 Routing Table - default - 5 entries      
D   ::/0 [90/1024640]              
     via FE80::16, Ethernet0/0 / ***** Local link соседа R16             
C   FD00::1030/124 [0/0]                
     via Ethernet0/0, directly connected              
L   FD00::1032/128 [0/0]               
     via Ethernet0/0, receive            
LC  FD00::FF01:20/128 [0/0]   / ****** Loopback 0                           
     via Loopback0, receive           
L   FF00::/8 [0/0]           
     via Null0, receive              

/ *************************** конец команд R32 ************************************** /
/ ***************************** диагностические команды R16 ************************** /                    

R16#show ip route             
Gateway of last resort is 0.0.0.0 to network 0.0.0.0                       
D*    0.0.0.0/0 is a summary, 01:11:58, Null0                  
      10.0.0.0/8 is variably subnetted, 22 subnets, 4 masks               
D        10.0.2.0/29 is a summary, 01:11:58, Null0 / **** Суммаризируем сети                     
D        10.0.2.0/30 [90/1029120] via 10.0.2.13, 01:11:56, Ethernet0/2              
D        10.0.2.4/30 [90/1029120] via 10.0.2.9, 01:11:56, Ethernet0/0                   
D        10.0.2.8/29 is a summary, 01:12:38, Null0  / **** Суммаризируем сети            
C        10.0.2.8/30 is directly connected, Ethernet0/0               
L        10.0.2.10/32 is directly connected, Ethernet0/0                 
C        10.0.2.12/30 is directly connected, Ethernet0/2              
L        10.0.2.14/32 is directly connected, Ethernet0/2            
C        10.0.2.16/30 is directly connected, Ethernet0/1            
L        10.0.2.17/32 is directly connected, Ethernet0/1               
C        10.0.2.20/30 is directly connected, Ethernet0/3              
L        10.0.2.22/32 is directly connected, Ethernet0/3                 
D        10.0.2.24/30 [90/1536000] via 10.0.2.18, 01:11:58, Ethernet0/1            
D        10.0.20.12/30 [90/1536000] via 10.0.2.18, 01:12:34, Ethernet0/1             
D        10.0.20.32/30 [90/1536000] via 10.0.2.18, 01:12:34, Ethernet0/1            
D        10.128.16.0/23 [90/1029120] via 10.0.2.13, 01:12:01, Ethernet0/2 / ***** Суммарный маршрут получает от SW9 и SW10                      
                        [90/1029120] via 10.0.2.9, 01:12:01, Ethernet0/0            
D        10.250.9.250/32 [90/1024640] via 10.0.2.13, 01:11:56, Ethernet0/2 /***** Lo0 от SW9                  
D        10.250.10.250/32 [90/1024640] via 10.0.2.9, 01:11:56, Ethernet0/0 /***** Lo0 от SW10                         
C        10.250.16.250/32 is directly connected, Loopback0                
D        10.250.17.250/32 [90/1029760] via 10.0.2.13, 01:11:58, Ethernet0/2 /***** Lo0 от R17                         
                          [90/1029760] via 10.0.2.9, 01:11:58, Ethernet0/0             
D        10.250.18.250/32 [90/1024640] via 10.0.2.18, 01:12:34, Ethernet0/1 /***** Lo0 от R18             
D        10.250.32.250/32 [90/1024640] via 10.0.2.21, 01:12:33, Ethernet0/3 /***** Lo0 от R32                             

R16#show ipv6 route            
IPv6 Routing Table - default - 24 entries             
D   ::/0 [5/1280]  / ****** суммарный маршрут для R32              
     via Null0, directly connected          
D   FD00::800/119 [90/1029120] / ****** суммарный маршрут получаем от SW9 и SW10                        
     via FE80::10, Ethernet0/0             
     via FE80::9, Ethernet0/2                 
D   FD00::1000/119 [5/1024000] / ****** суммаризируем транзитные сети                         
     via Null0, directly connected               
C   FD00::1000/124 [0/0]                   
     via Ethernet0/0, directly connected                
L   FD00::1001/128 [0/0]                    
     via Ethernet0/0, receive                 
C   FD00::1020/124 [0/0]                    
     via Ethernet0/2, directly connected                 
L   FD00::1021/128 [0/0]                    
     via Ethernet0/2, receive                   
C   FD00::1030/124 [0/0]                
     via Ethernet0/3, directly connected                  
L   FD00::1031/128 [0/0]                 
     via Ethernet0/3, receive               
D   FD00::1100/124 [90/1029120]                 
     via FE80::9, Ethernet0/2               
D   FD00::1120/124 [90/1029120]                
     via FE80::10, Ethernet0/0                 
C   FD00::1200/124 [0/0]               
     via Ethernet0/1, directly connected                
L   FD00::1202/128 [0/0]                   
     via Ethernet0/1, receive                   
D   FD00::1210/124 [90/1536000]                  
     via FE80::18, Ethernet0/1               
D   FD00::1830/124 [90/1536000]                  
     via FE80::18, Ethernet0/1                
D   FD00::1A30/124 [90/1536000]                 
     via FE80::18, Ethernet0/1               
D   FD00::FF01:0/120 [5/1280]  / *********** суммаризируем Lo SW9 SW 10 R16 R17 R32             
     via Null0, directly connected             
D   FD00::FF01:9/128 [90/1024640]               
     via FE80::9, Ethernet0/2               
D   FD00::FF01:A/128 [90/1024640]              
     via FE80::10, Ethernet0/0              
LC  FD00::FF01:10/128 [0/0]               
     via Loopback0, receive             
D   FD00::FF01:11/128 [90/1029760]                
     via FE80::10, Ethernet0/0               
     via FE80::9, Ethernet0/2              
D   FD00::FF01:12/128 [90/1024640]              
     via FE80::18, Ethernet0/1              
D   FD00::FF01:20/128 [90/1024640]              
     via FE80::32, Ethernet0/3          
L   FF00::/8 [0/0]               
     via Null0, receive               
/ ***************************** диагностические команды R16 end ************************** /                      
/ ***************************** диагностические команды R18 ************************** /                       
R18#show ip route                    
Gateway of last resort is 10.0.20.34 to network 0.0.0.0             

S*    0.0.0.0/0 [1/0] via 10.0.20.34              
                [1/0] via 10.0.20.14                    
      10.0.0.0/8 is variably subnetted, 18 subnets, 4 masks                
D        10.0.2.0/29 [90/1536000] via 10.0.2.25, 01:27:44, Ethernet0/1  /***** Суммарный маршрут                           
D        10.0.2.8/29 [90/1536000] via 10.0.2.17, 01:27:40, Ethernet0/0  /***** Суммарный маршрут                  
C        10.0.2.16/30 is directly connected, Ethernet0/0              
L        10.0.2.18/32 is directly connected, Ethernet0/0                
D        10.0.2.20/30 [90/1536000] via 10.0.2.17, 01:27:45, Ethernet0/0            
C        10.0.2.24/30 is directly connected, Ethernet0/1             
L        10.0.2.26/32 is directly connected, Ethernet0/1            
C        10.0.20.12/30 is directly connected, Ethernet0/2          
L        10.0.20.13/32 is directly connected, Ethernet0/2           
C        10.0.20.32/30 is directly connected, Ethernet0/3          
L        10.0.20.33/32 is directly connected, Ethernet0/3              
D        10.128.16.0/23 [90/1541120] via 10.0.2.25, 01:27:45, Ethernet0/1   /***** Суммарный маршрут                  
                        [90/1541120] via 10.0.2.17, 01:27:45, Ethernet0/0             
D        10.250.9.250/32 [90/1536640] via 10.0.2.25, 01:27:45, Ethernet0/1  /******* Lo0 все          
                         [90/1536640] via 10.0.2.17, 01:27:45, Ethernet0/0               
         10.250.10.250/32 [90/1536640] via 10.0.2.25, 01:39:40, Ethernet0/1                                    
                          [90/1536640] via 10.0.2.17, 01:27:45, Ethernet0/0                 
D        10.250.16.250/32 [90/1024640] via 10.0.2.17, 01:27:45, Ethernet0/0              
D        10.250.17.250/32 [90/1024640] via 10.0.2.25, 01:27:45, Ethernet0/1             
C        10.250.18.250/32 is directly connected, Loopback0                
D        10.250.32.250/32 [90/1536640] via 10.0.2.17, 01:27:45, Ethernet0/0              

R18#show ipv6 route              
D   FD00::800/119 [90/1541120] /***** Суммарный маршрут пользовательские сети                                
     via FE80::16, Ethernet0/0  /**** от R16             
     via FE80::17, Ethernet0/1  /**** от R17            
D   FD00::1000/119 [90/1536000] /***** Суммарный маршрут транзитные сети                                           
     via FE80::17, Ethernet0/1             
     via FE80::16, Ethernet0/0               
C   FD00::1200/124 [0/0]            
     via Ethernet0/0, directly connected              
L   FD00::1201/128 [0/0]              
     via Ethernet0/0, receive           
C   FD00::1210/124 [0/0]             
     via Ethernet0/1, directly connected             
L   FD00::1211/128 [0/0]           
     via Ethernet0/1, receive           
C   FD00::1830/124 [0/0]          
     via Ethernet0/2, directly connected            
L   FD00::1832/128 [0/0]              
     via Ethernet0/2, receive              
C   FD00::1A30/124 [0/0]             
     via Ethernet0/3, directly connected                 
L   FD00::1A32/128 [0/0]             
     via Ethernet0/3, receive             
D   FD00::FF01:0/120 [90/1024640] /***** Суммарный маршрут Lo0 SW9, SW10, R16, R17, R32                                
     via FE80::17, Ethernet0/1            
     via FE80::16, Ethernet0/0           
LC  FD00::FF01:12/128 [0/0]             
     via Loopback0, receive           
L   FF00::/8 [0/0]              
     via Null0, receive                      
/ ***************************** диагностические команды R18 end************************** /                       





