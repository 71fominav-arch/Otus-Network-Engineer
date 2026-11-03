## Лабораторная работа №5
### Задание.
1. В офисе С.-Петербург настроить EIGRP.
2. R32 получает только маршрут по умолчанию.
3. R16-17 анонсируют только суммарные префиксы.
4. Использовать EIGRP named-mode для настройки сети.

![Офис Питербург](./Piter.png)               

/***************** Настройки роутера R16 **********************/                     
router eigrp Piter            
 !
 address-family ipv4 unicast autonomous-system 100           
  !          
  af-interface default         
   passive-interface        
  exit-af-interface        
  !       
  af-interface Ethernet0/1           
   summary-address 10.0.2.0 255.255.255.248           
   summary-address 10.0.2.8 255.255.255.248            
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
   summary-address 0.0.0.0 0.0.0.0              
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
   summary-address FD00::1000/119             
   summary-address FD00::FF01:0/120           
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
   summary-address ::/0               
   no passive-interface              
  exit-af-interface              
  !             
  topology base             
  exit-af-topology                  
  eigrp router-id 10.250.16.250             
 exit-address-family          
!                     
