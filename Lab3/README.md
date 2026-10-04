# Лабораторная работа №3           

1. Маршрутизаторы R14-R15 находятся в зоне 0 - backbone.           
2. Маршрутизаторы R12-R13 находятся в зоне 10. Дополнительно к маршрутам должны получать маршрут по умолчанию.            
3. Маршрутизатор R19 находится в зоне 101 и получает только маршрут по умолчанию.            
4. Маршрутизатор R20 находится в зоне 102 и получает все маршруты, кроме маршрутов до сетей зоны 101.            
5. Настройка для IPv6 повторяет логику IPv4.            

![Схема OSPF](./Moscow_ospf.png)

## 1. Маршрутизаторы R14-R15 находятся в зоне 0 - backbone.    
Маршрутизаторы R14-R15 являются ABR, то есть часть интерфейсов находятся в других зонах.       

         
| Маршрутизатор    | E0/0 | E0/1 | E0/2 | E0/3 | Loopback 0 |              
|------------------|------|------|------|------|------------|             
| R14 | area 10 | area 0 | area 0 | area 101 | area 0 |             
| R15 | area 10 | area 0 | area 0 | area 102 | area 0 |                        

                           
Настройки на R14                         
                    
### Пример конфигурации R14           
/ ****** R14 *********/                
router ospf 10 /* включаем процесс */                                   
 router-id 10.250.14.250                    
 passive-interface default /* по умолчанию все интерфейсы в пассив режиме, не ищут соседей */               
 no passive-interface Ethernet0/0 /* поиск соседей на интерфейсе e0/0 */                         
 no passive-interface Ethernet0/1           
 no passive-interface Ethernet0/3                   
 !         
interface Loopback0             
 ip address 10.250.14.250 255.255.255.255                  
 ip ospf 10 area 0            
!
interface Ethernet0/0          
 description to_R12         
 ip address 10.0.0.38 255.255.255.252          
 ip ospf 10 area 10           
!             
interface Ethernet0/1            
 description to_R15             
 ip address 10.0.0.45 255.255.255.252           
 ip ospf 10 area 0           
!            
interface Ethernet0/2           
 description to_R22              
 ip address 10.0.16.1 255.255.255.252           
 ip ospf 10 area 0            
!              
interface Ethernet0/3           
 description to_R19          
 ip address 10.0.0.34 255.255.255.252             
 ip ospf 10 area 101           
!                      
/ ********* end R14 ************/                  


## 2. Маршрутизаторы R12-R13 находятся в зоне 10. Дополнительно к маршрутам должны получать маршрут по умолчанию.         

Маршрутизаторы R12-R13, а также коммутаторы SW4, SW5 находятся в зоне 10. Тип normal. R14, R15 должны передавать маршрут по умолчанию.         

Все устройства R12, R13, SW4, SW5 полностью в зоне 10, ABR для зоны 10 R14,R15.
Пример настройки R12          
/ **************** R12 ************** /
                    
interface Loopback0                 
 ip address 10.250.12.250 255.255.255.255              
 ip ospf 10 area 10            
!                
interface Ethernet0/0             
 description to_SW4                
 ip address 10.0.0.18 255.255.255.252               
 ip ospf 10 area 10              
!                 
interface Ethernet0/1                
 description to_SW5               
 ip address 10.0.0.26 255.255.255.252           
 ip ospf 10 area 10           
!              
interface Ethernet0/2          
 description to_R14             
 ip address 10.0.0.37 255.255.255.252           
 ip ospf 10 area 10           
!              
interface Ethernet0/3         
 description to_R13            
 ip address 10.0.0.41 255.255.255.252            
 ip ospf 10 area 10             
 !           
 router ospf 10           
 router-id 10.250.12.250            
 passive-interface default          
 no passive-interface Ethernet0/0         
 no passive-interface Ethernet0/1           
 no passive-interface Ethernet0/2            
 no passive-interface Ethernet0/3          
!            
/ *********** end R12 *************** /                    















