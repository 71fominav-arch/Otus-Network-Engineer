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
    













 
   
