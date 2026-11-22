# Лабораторная работа №7

## Задание:      
1. Настроите iBGP в офисом Москва между маршрутизаторами R14 и R15.        
2. Настроите iBGP в провайдере Триада, с использованием RR.        
3. Настройте офиса Москва так, чтобы приоритетным провайдером стал Ламас.         
4. Настройте офиса С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно.           
5. Все сети в лабораторной работе должны иметь IP связность.           


### 1. Настроить iBGP в офисе Москва между маршрутизаторами R14 и R15.

 / ********************************** R14 ******************************** /                    
  router bgp 1001            
    bgp router-id 10.250.14.250            
        neighbor 10.250.15.250 remote-as 1001         
        neighbor 10.250.15.250 update-source Loopback0        
        neighbor 10.250.15.250 next-hop-self  (сеть 10.0.16.0/30 не анонсирована, для соседа ставим next hop себя)                        
 / ****************************** end R14 ******************************** /                  

  / ********************************** R15 ******************************** /                    
  router bgp 1001            
    bgp router-id 10.250.15.250            
        neighbor 10.250.14.250 remote-as 1001        
        neighbor 10.250.14.250 update-source Loopback0       
        neighbor 10.250.14.250 next-hop-self  (сеть 10.0.18.0/30 не анонсирована, для соседа ставим next hop себя)                         
 / ****************************** end R15 ******************************** /               

### 2. Настроите iBGP в провайдере Триада, с использованием RR.                 

     R23 и R25 сделаем RR.
     R24 и R26 клиенты.

 / ********************************** R23 ******************************** /                    
 приводятся только настройки для iBGP               
  router bgp 520               
       bgp router-id 10.250.23.250             
       bgp cluster-id 10.250.250.250    / ***  для предотвращения петель между двумя RR *** /                
       network 10.250.23.250 mask 255.255.255.255                    
       neighbor RR-CLIENTS peer-group   / *** настройки peer group для клиентов RR  *** /            
       neighbor RR-CLIENTS remote-as 520  / *** настройки peer group для клиентов RR  *** /           
       neighbor RR-CLIENTS update-source Loopback0  / *** настройки peer group для клиентов RR   *** /           
       neighbor RR-CLIENTS route-reflector-client  / *** настройки peer group для клиентов RR  *** /           
       neighbor RR-CLIENTS next-hop-self  / *** настройки peer group для клиентов RR в атрибуте next-hop для внешних сетей ставим свой IP Lo0 *** /           
       neighbor 10.250.24.250 peer-group RR-CLIENTS  / *** клиент RR *** /                  
       neighbor 10.250.25.250 remote-as 520  / *** сосед такой же RR *** /                   
       neighbor 10.250.25.250 update-source Loopback0                
       neighbor 10.250.25.250 next-hop-self                
       neighbor 10.250.26.250 peer-group RR-CLIENTS   / *** клиент RR *** /                  
                        
 / ****************************** end R23 ******************************** /               

 / ********************************** R25 ******************************** /                    
  приводятся только настройки для iBGP               
  router bgp 520             
        bgp router-id 10.250.25.250             
        bgp cluster-id 10.250.250.250            
        network 10.250.25.250 mask 255.255.255.255             
        neighbor RR-CLIENTS peer-group               
        neighbor RR-CLIENTS remote-as 520                
        neighbor RR-CLIENTS update-source Loopback0                    
        neighbor RR-CLIENTS route-reflector-client                     
        neighbor RR-CLIENTS next-hop-self                      
        neighbor 10.250.23.250 remote-as 520                    
        neighbor 10.250.23.250 update-source Loopback0                      
        neighbor 10.250.23.250 next-hop-self                        
        neighbor 10.250.24.250 peer-group RR-CLIENTS                   
        neighbor 10.250.26.250 peer-group RR-CLIENTS                       
                       
 / ****************************** end R25 ******************************** /               
    
 / ********************************** R24 ******************************** /                    
  приводятся только настройки для iBGP               
  router bgp 520                  
        bgp router-id 10.250.24.250                 
        bgp log-neighbor-changes                  
        network 10.250.24.250 mask 255.255.255.255               
        neighbor 10.250.23.250 remote-as 520 / *** RR 1 *** /                
        neighbor 10.250.23.250 update-source Loopback0                
        neighbor 10.250.23.250 next-hop-self                
        neighbor 10.250.25.250 remote-as 520 / *** RR 2 *** /                               
        neighbor 10.250.25.250 update-source Loopback0                  
        neighbor 10.250.25.250 next-hop-self                                      
 / ****************************** end R24 ******************************** /               

 / ********************************** R26 ******************************** /                    
  приводятся только настройки для iBGP               
  router bgp 520                         
        bgp router-id 10.250.26.250              
        bgp log-neighbor-changes               
        network 10.250.26.250 mask 255.255.255.255                 
        neighbor 10.250.23.250 remote-as 520   / *** RR 1 *** /           
        neighbor 10.250.23.250 update-source Loopback0              
        neighbor 10.250.23.250 next-hop-self             
        neighbor 10.250.25.250 remote-as 520  / *** RR 2 *** /          
        neighbor 10.250.25.250 update-source Loopback0           
        neighbor 10.250.25.250 next-hop-self                       
 / ****************************** end R26 ******************************** /               

### 3. Настройте офиса Москва так, чтобы приоритетным провайдером стал Ламас.        

Для  того, чтобы маршрут через Ламас стал приоритетным, меняем атрибут local-preference. Он по умолчанию 100, ставим 200. Внутри одной AS передается.         
Для всех входящих маршрутов от peer 10.0.18.2 ставим local-preference 200.               

  / ********************************** R15 ******************************** /                    
  route-map LP-200-ALL permit 10            
  set local-preference 200            

  router bgp 1001            
    bgp router-id 10.250.15.250            
         neighbor 10.0.18.2 route-map LP-200-ALL in            
 / ****************************** end R15 ******************************** /               

/ *************** Диагностика R14 *************************************** /
  R14#show ip bgp neighbors 10.250.15.250 routes          
BGP table version is 66, local router ID is 10.250.14.250         
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,         
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,          
              x best-external, a additional-path, c RIB-compressed,         
Origin codes: i - IGP, e - EGP, ? - incomplete         
RPKI validation codes: V valid, I invalid, N Not found              
        
 / **    Network          Next Hop            Metric LocPrf Weight Path           
/ * i 10.128.0.0/20    10.250.15.250            0    100      0 i          
/ *>i 10.128.16.0/20   10.250.15.250            0    200      0 301 520 2042 i           
/ r>i 10.250.15.250/32 10.250.15.250            0    100      0 i           
/ *>i 10.250.18.250/32 10.250.15.250            0    200      0 301 520 2042 i  / ***** LocPrf 200  *** /                 
/ *>i 10.250.21.250/32 10.250.15.250            0    200      0 301 i             
/ *>i 10.250.22.250/32 10.250.15.250            0    200      0 301 101 i            
/ *>i 10.250.23.250/32 10.250.15.250            0    200      0 301 520 i           
/ *>i 10.250.24.250/32 10.250.15.250            0    200      0 301 520 i          
/ *>i 10.250.25.250/32 10.250.15.250            0    200      0 301 520 i              
/ *>i 10.250.26.250/32 10.250.15.250            0    200      0 301 520 i                  
   ##### из вывода команды видно, что внешние для AS 1001 сети отправляем соседу с измененным атрибутом  LocPrf 200              
 R14#show ip bgp       
BGP table version is 66, local router ID is 10.250.14.250           
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,           
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,        
              x best-external, a additional-path, c RIB-compressed,        
Origin codes: i - IGP, e - EGP, ? - incomplete          
RPKI validation codes: V valid, I invalid, N Not found          
               
/ **    Network          Next Hop            Metric LocPrf Weight Path             
/ * i 10.128.0.0/20    10.250.15.250            0    100      0 i                
/ *>                   0.0.0.0                  0         32768 i / *** приоритет сам r14                     
/ *>i 10.128.16.0/20   10.250.15.250            0    200      0 301 520 2042 i / *** приоритет  r15, next hop Lo0 R15, LocPrf 200                                  
/ *                    10.0.16.2                              0 101 520 2042 i             
/ *>  10.250.14.250/32 0.0.0.0                  0         32768 i           
/ r>i 10.250.15.250/32 10.250.15.250            0    100      0 i                
/ *>i 10.250.18.250/32 10.250.15.250            0    200      0 301 520 2042 i              
/ *                    10.0.16.2                              0 101 520 2042 i           
/ *>i 10.250.21.250/32 10.250.15.250            0    200      0 301 i             
/ *                    10.0.16.2                              0 101 301 i           
/ *>i 10.250.22.250/32 10.250.15.250            0    200      0 301 101 i           
/ *                    10.0.16.2                0             0 101 i             
/ *>i 10.250.23.250/32 10.250.15.250            0    200      0 301 520 i             
/ *                    10.0.16.2                              0 101 520 i                       
/ *>i 10.250.24.250/32 10.250.15.250            0    200      0 301 520 i           
/ *                    10.0.16.2                              0 101 520 i            
/ *>i 10.250.25.250/32 10.250.15.250            0    200      0 301 520 i             
/ *                    10.0.16.2                              0 101 520 i             
/ *>i 10.250.26.250/32 10.250.15.250            0    200      0 301 520 i                     
/ *                    10.0.16.2                              0 101 520 i              
  
##### Для внешних маршрутов приоритет через R15 и Ламас. Задание выполнено.                    
 / *************** end Диагностика R14 *************************************** /                
/ *************** Диагностика R15 *************************************** /                
R15#show ip bgp                
BGP table version is 65, local router ID is 10.250.15.250               
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,            
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,             
              x best-external, a additional-path, c RIB-compressed,         
Origin codes: i - IGP, e - EGP, ? - incomplete          
RPKI validation codes: V valid, I invalid, N Not found          

/ **   Network          Next Hop            Metric LocPrf Weight Path             
 * i 10.128.0.0/20    10.250.14.250            0    100      0 i         
 *>                   0.0.0.0                  0         32768 i           
 *>  10.128.16.0/20   10.0.18.2                     200      0 301 520 2042 i          
 r>i 10.250.14.250/32 10.250.14.250            0    100      0 i          
 *>  10.250.15.250/32 0.0.0.0                  0         32768 i         
 *>  10.250.18.250/32 10.0.18.2                     200      0 301 520 2042 i          
 *>  10.250.21.250/32 10.0.18.2                0    200      0 301 i          
 *>  10.250.22.250/32 10.0.18.2                     200      0 301 101 i           
 *>  10.250.23.250/32 10.0.18.2                     200      0 301 520 i           
 *>  10.250.24.250/32 10.0.18.2                     200      0 301 520 i         
 *>  10.250.25.250/32 10.0.18.2                     200      0 301 520 i        
 *>  10.250.26.250/32 10.0.18.2                     200      0 301 520 i                 
   ##### Здесь только маршруты через Ламас                      
   ##### Если уронить интерфейс в сторону Ламас, то появился маршрут через R14 R22, резервирование работает.                                           

R15#show ip bgp                  
BGP table version is 81, local router ID is 10.250.15.250             
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,            
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,           
              x best-external, a additional-path, c RIB-compressed,          
Origin codes: i - IGP, e - EGP, ? - incomplete          
RPKI validation codes: V valid, I invalid, N Not found         
/ **   Network          Next Hop            Metric LocPrf Weight Path            
 * i 10.128.0.0/20    10.250.14.250            0    100      0 i           
 *>                   0.0.0.0                  0         32768 i           
 *>i 10.128.16.0/20   10.250.14.250            0    100      0 101 520 2042 i          
 r>i 10.250.14.250/32 10.250.14.250            0    100      0 i           
 *>  10.250.15.250/32 0.0.0.0                  0         32768 i           
 *>i 10.250.18.250/32 10.250.14.250            0    100      0 101 520 2042 i           
 *>i 10.250.21.250/32 10.250.14.250            0    100      0 101 301 i         
 *>i 10.250.22.250/32 10.250.14.250            0    100      0 101 i            
 *>i 10.250.23.250/32 10.250.14.250            0    100      0 101 520 i            
 *>i 10.250.24.250/32 10.250.14.250            0    100      0 101 520 i             
 *>i 10.250.25.250/32 10.250.14.250            0    100      0 101 520 i          
 *>i 10.250.26.250/32 10.250.14.250            0    100      0 101 520 i                
 / *************** end Диагностика R15 *************************************** /                 

### 4. Настройте офиса С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно.           
Если посмотреть команду show ip bgp  10.250.15.250, то видно что этот маршрут приходит от двух соседов и атрибуты одинаковые,
кроме AS PATH: от R24 520 301 1001, от R26 520 101 1001                               
Поскольку этот атрибут отличается, нужно ввести команду для игнорирования его. И команду для использования двух маршрутов одновременно.        
/ ****************** Настройка R18 ********************************************* /
   router bgp 2042                            
    bgp router-id 10.250.18.250                           
    bgp bestpath as-path multipath-relax / *** не сравнивать номера AS, при этом количество AS в цепочке должно быть одинаковым *** /            
    maximum-paths 2  / **** использовать до двух маршрутов при возможности, можно ставить до 16, 32 зависит от моделей ******* /

/ ****************** Настройка R18 ********************************************* /                   
/ *************** Диагностика R18 ********************************************* /              
R18#show ip bgp  10.250.15.250             
BGP routing table entry for 10.250.15.250/32, version 44                
Paths: (2 available, best #2, table default)            
Multipath: eBGP          
  Advertised to update-groups:          
     1          
  Refresh Epoch 1        
  520 301 1001           
    10.0.20.14 from 10.0.20.14 (10.250.24.250)          
      Origin IGP, localpref 100, valid, external, multipath(oldest)            
      rx pathid: 0, tx pathid: 0          
  Refresh Epoch 1          
  520 101 1001             
    10.0.20.34 from 10.0.20.34 (10.250.26.250)              
      Origin IGP, localpref 100, valid, external, multipath, best          
      rx pathid: 0, tx pathid: 0x0          
 R18#show ip bgp            
BGP table version is 44, local router ID is 10.250.18.250             
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,          
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,          
              x best-external, a additional-path, c RIB-compressed,         
Origin codes: i - IGP, e - EGP, ? - incomplete        
RPKI validation codes: V valid, I invalid, N Not found         

/ **   Network          Next Hop            Metric LocPrf Weight Path       
 *m  10.128.0.0/20    10.0.20.14                             0 520 301 1001 i           
 *>                   10.0.20.34                             0 520 101 1001 i           
 *>  10.128.16.0/20   0.0.0.0                  0         32768 i            
 *m  10.250.14.250/32 10.0.20.14                             0 520 301 1001 i            
 *>                   10.0.20.34                             0 520 101 1001 i             
 *m  10.250.15.250/32 10.0.20.14                             0 520 301 1001 i             
 *>                   10.0.20.34                             0 520 101 1001 i             
 *>  10.250.18.250/32 0.0.0.0                  0         32768 i               
 *m  10.250.21.250/32 10.0.20.34                             0 520 301 i              
 *>                   10.0.20.14                             0 520 301 i            
 *m  10.250.22.250/32 10.0.20.34                             0 520 101 i             
 *>                   10.0.20.14                             0 520 101 i              
 *m  10.250.23.250/32 10.0.20.34                             0 520 i                
 *>                   10.0.20.14                             0 520 i                
 *m  10.250.24.250/32 10.0.20.34                             0 520 i                  
 *>                   10.0.20.14               0             0 520 i             
 *m  10.250.25.250/32 10.0.20.34                             0 520 i             
 *>                   10.0.20.14                             0 520 i             
 *m  10.250.26.250/32 10.0.20.14                             0 520 i                      
 *>                   10.0.20.34               0             0 520 i      

 ##### Появился знак m multipath. Задание выполнено.
/ *************** end Диагностика R18 ********************************************* /             

### 5. Все сети в лабораторной работе должны иметь IP связность. 

По итогу работ, анонсируемые сети 10.250.RX.250, 10.128.0.0/20, 10.128.16.0/20 видны. Хосты доступны между собой.               
                      
[Конфигурации устройств](./Conf/README.md)                      








 
   
