# Задание - настроить протокол IS-IS для IPv4 и IPv6 в ISP Триада.            

#### Настроите IS-IS в ISP Триада.
#### R23 и R25 находятся в зоне 2222.
#### R24 находится в зоне 24.
#### R26 находится в зоне 26.

### Схема сети Триада            
![Триада сеть](./Triada.png)


### Пояснения к схеме             
   Всего 4 маршрутизатора    
   R23 net 49.2222.0102.5002.3250.00          
   R24 net 49.0024.0102.5024.2500.00          
   R25 net 49.2222.0102.5002.5250.00            
   R26 net 49.0026.0102.5002.6250.00              
   Всего три зоны и 4 маршрутизатора                  
   Все 4 маршрутизатора настроим на Level-2 маршрутизацию.              
   На интерфейсах включим парольную защиту.                

### Настройки маршрутизатора R23

1. IPv4 уже настроен.
2. Ipv6            
   2.1 Включим Ipv6          
       ipv6 unicast-routing           
   2.2 Настройка интерфейсов           
      interface Ethernet0/0         
          ip address 10.0.16.6 255.255.255.252          
          ipv6 address FD00::2300:1/112            
      !         
      interface Ethernet0/1         
          ip address 10.0.20.1 255.255.255.252            
          ipv6 address FD00::2301:1/112           
      !                  
      interface Ethernet0/2           
          ip address 10.0.20.6 255.255.255.252             
          ipv6 address FD00::2302:1/112
3. Включим ISIS                      
      router isis 100               
      net 49.2222.0102.5002.3250.00               
      is-type level-2-only                     
      authentication mode md5           
      metric-style wide              
      log-adjacency-changes            
      address-family ipv6
                        
5. Настройка ISIS на интерфейсах                 

      interface Loopback0       
      ip router isis 100           
      !             
      interface Ethernet0/0          
      ip router isis 100            
      ipv6 router isis 100              
      !            
      interface Ethernet0/1                                  
      ip router isis 100      /* включить протокол маршрутизации на интерфейсе         
      ipv6 router isis 100   /* включить протокол маршрутизации на интерфейсе                       
      isis password cisco123 level-2  /* авторизация с соседом                
      !                  
      interface Ethernet0/2               
      ip router isis 100  /* включить протокол маршрутизации на интерфейсе                      
      ipv6 router isis 100 /* включить протокол маршрутизации на интерфейсе                             
      isis password cisco123 level-2                    
  





