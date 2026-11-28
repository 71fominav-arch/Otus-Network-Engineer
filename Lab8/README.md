# Лабораторная работа №8

## Задание
1. Настроить фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика(As-path).
2. Настроить фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика(Prefix-list).
3. Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по умолчанию.
4. Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по умолчанию и префикс офиса С.-Петербург.               

### 1. Настроить фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика(As-path).                            

Для того, чтобы не появлялось транзитного трафика в офисе Москва, нужно чтобы R14 и R15 отдавали префиксы только своей AS.                      

/ ************** R14 ************************************** /                               
ip as-path access-list 100 permit ^$                   
 router bgp 1001                   
 neighbor 10.0.16.2 filter-list 100 out                  
end                     
/ **************** end R14 ******************************* /                         
/ **************** R15 *********************************** /                       
ip as-path access-list 100 permit ^$                   
 router bgp 1001                           
 neighbor 10.0.18.2 filter-list 100 out             
 end               
/ **************** end R15 ******************************* /                
/ ******************** Диагностика  R14 ****************** /                   
#### До filter-list
R14#show ip bgp neighbors 10.0.16.2 advertised-routes                     
BGP table version is 19, local router ID is 10.250.14.250                      
     Network          Next Hop            Metric LocPrf Weight Path                         
 r>i 0.0.0.0          10.250.15.250            0    200      0 301 i                    
 *>  10.128.0.0/20    0.0.0.0                  0         32768 i                   
 *>i 10.128.16.0/20   10.250.15.250            0    200      0 301 520 2042 i                
 *>  10.250.14.250/32 0.0.0.0                  0         32768 i                     
 r>i 10.250.15.250/32 10.250.15.250            0    100      0 i                         
 *>i 10.250.18.250/32 10.250.15.250            0    200      0 301 520 2042 i                   
 *>i 10.250.21.250/32 10.250.15.250            0    200      0 301 i                  
 *>i 10.250.22.250/32 10.250.15.250            0    200      0 301 101 i                    
 *>i 10.250.23.250/32 10.250.15.250            0    200      0 301 520 i                   
 *>i 10.250.24.250/32 10.250.15.250            0    200      0 301 520 i                  
 *>i 10.250.25.250/32 10.250.15.250            0    200      0 301 520 i                 
 *>i 10.250.26.250/32 10.250.15.250            0    200      0 301 520 i                      
                        
##### Отдаем сети не только свои, но и транзитные.                    
                           
#### После применения
R14#show ip bgp neighbors 10.0.16.2 advertised-routes            
BGP table version is 19, local router ID is 10.250.14.250           
     Network          Next Hop            Metric LocPrf Weight Path                        
 *>  10.128.0.0/20    0.0.0.0                  0         32768 i             
 *>  10.250.14.250/32 0.0.0.0                  0         32768 i             
 r>i 10.250.15.250/32 10.250.15.250            0    100      0 i          
                    
Total number of prefixes 3                            
                 
##### Отдаем сети только своей AS                          

/ ******************** end R14 *************************** /                       
### 2. Настроить фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика(Prefix-list).

Для того чтобы решить проблему с помощью Prefix-list, записываем в него только сети которые мы отдаем.                       

/ ***************************** R18 ******************************* /                               
ip prefix-list only-local-net seq 100 permit 10.250.18.250/32                       
  ip prefix-list only-local-net seq 101 permit 10.128.16.0/20                  
  route-map filter_net_out permit 100                                    
   match ip address prefix-list only-local-net                              
   exit                     
   router bgp 2042                         
   neighbor 10.0.20.14 route-map filter_net_out out                         
   neighbor 10.0.20.34 route-map filter_net_out out                         
   end                   
/ ************************* end R18 ******************************* /                    
/ ************************* Диагностика R18 *********************** /                        
R18#show ip bgp neighbors 10.0.20.14 advertised-routes               
BGP table version is 18, local router ID is 10.250.18.250                         
     Network          Next Hop            Metric LocPrf Weight Path              
 *>  10.128.0.0/20    10.0.20.14                             0 520 301 1001 i         
 *>  10.128.16.0/20   0.0.0.0                  0         32768 i             
 *>  10.250.14.250/32 10.0.20.14                             0 520 301 1001 i             
 *>  10.250.15.250/32 10.0.20.14                             0 520 301 1001 i            
 *>  10.250.18.250/32 0.0.0.0                  0         32768 i                
 *>  10.250.21.250/32 10.0.20.14                             0 520 301 i              
 *>  10.250.22.250/32 10.0.20.14                             0 520 101 i             
 *>  10.250.23.250/32 10.0.20.14                             0 520 i             
 *>  10.250.24.250/32 10.0.20.14               0             0 520 i             
 *>  10.250.25.250/32 10.0.20.14                             0 520 i               
 *>  10.250.26.250/32 10.0.20.34               0             0 520 i                  
                     
Total number of prefixes 11                    
R18#show ip bgp neighbors 10.0.20.34 advertised-routes                          
BGP table version is 18, local router ID is 10.250.18.250                              
     Network          Next Hop            Metric LocPrf Weight Path                 
 *>  10.128.0.0/20    10.0.20.14                             0 520 301 1001 i              
 *>  10.128.16.0/20   0.0.0.0                  0         32768 i              
 *>  10.250.14.250/32 10.0.20.14                             0 520 301 1001 i               
 *>  10.250.15.250/32 10.0.20.14                             0 520 301 1001 i             
 *>  10.250.18.250/32 0.0.0.0                  0         32768 i              
 *>  10.250.21.250/32 10.0.20.14                             0 520 301 i               
 *>  10.250.22.250/32 10.0.20.14                             0 520 101 i              
 *>  10.250.23.250/32 10.0.20.14                             0 520 i              
 *>  10.250.24.250/32 10.0.20.14               0             0 520 i               
 *>  10.250.25.250/32 10.0.20.14                             0 520 i                  
 *>  10.250.26.250/32 10.0.20.34               0             0 520 i                
                
Total number of prefixes 11              
##### Отдаем не только свои сети, но и транзитные.                      
После применения route-map                    
R18#show ip bgp neighbors 10.0.20.14 advertised-routes                 
BGP table version is 18, local router ID is 10.250.18.250               
     Network          Next Hop            Metric LocPrf Weight Path                    
 *>  10.128.16.0/20   0.0.0.0                  0         32768 i                  
 *>  10.250.18.250/32 0.0.0.0                  0         32768 i              
Total number of prefixes 2                  
R18#clear ip bgp 10.0.20.34 soft                  
R18#show ip bgp neighbors 10.0.20.34 advertised-routes                           
BGP table version is 18, local router ID is 10.250.18.250                            
     Network          Next Hop            Metric LocPrf Weight Path                   
 *>  10.128.16.0/20   0.0.0.0                  0         32768 i                   
 *>  10.250.18.250/32 0.0.0.0                  0         32768 i                 
                            
Total number of prefixes 2                      
##### Отдаем только свои сети                 
                       
/ ************************* end R18 ******************************* /                              

### 3. Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по умолчанию.

Сделать префикс лист с единственной записью 0.0.0.0/0                            
анонсировать маршрут по умолчанию в сторону соседа и применить префикс лист.                     
                                 
ip prefix-list only-default-out seq 100 permit 0.0.0.0/0                      
   route-map to_moscow_out permit 100                   
    match ip address prefix-list only-default-out                
   exit                    
router bgp 101                          
   neighbor 10.0.16.1 default-originate               
   neighbor 10.0.16.1 route-map to_moscow_out out             
end                     

/ ************************************ R22 диагностика *************************** /
##### До 
R22#show ip bgp neighbors 10.0.16.1 advertised-routes                
BGP table version is 18, local router ID is 10.250.22.250                                 
Originating default network 0.0.0.0          
     Network          Next Hop            Metric LocPrf Weight Path               
 *>  10.128.16.0/20   10.0.16.6                              0 520 2042 i            
 *>  10.250.18.250/32 10.0.16.6                              0 520 2042 i          
 *>  10.250.21.250/32 10.0.18.5                0             0 301 i            
 *>  10.250.22.250/32 0.0.0.0                  0         32768 i          
 *>  10.250.23.250/32 10.0.16.6                0             0 520 i           
 *>  10.250.24.250/32 10.0.16.6                              0 520 i           
 *>  10.250.25.250/32 10.0.16.6                              0 520 i           
 *>  10.250.26.250/32 10.0.16.6                              0 520 i          
Total number of prefixes 8                  
##### После
R22#show ip bgp neighbors 10.0.16.1 advertised-routes              
BGP table version is 18, local router ID is 10.250.22.250          
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,         
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,           
              x best-external, a additional-path, c RIB-compressed,        
Origin codes: i - IGP, e - EGP, ? - incomplete         
RPKI validation codes: V valid, I invalid, N Not found          

Originating default network 0.0.0.0            

     Network          Next Hop            Metric LocPrf Weight Path            

Total number of prefixes 0           
##### Только маршрут по умолчанию                     

/ ************************************ end R22 *********************************** /                    

### 4. Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по умолчанию и префикс офиса С.-Петербург.           

   ip prefix-list Piter-net seq 100 permit 10.250.18.250/32                 
   ip prefix-list Piter-net seq 101 permit 10.128.16.0/20               
   ip prefix-list only-default-out seq 100 permit 0.0.0.0/0               
                   
   route-map to_moscow_out permit 100             
    match ip address prefix-list Piter-net             
   exit              
   route-map to_moscow_out permit 110              
    match ip address prefix-list only-default-out               
   exit                                  
   neighbor 10.0.18.1 default-originate               
   neighbor 10.0.18.1 route-map to_moscow_out out                 

/ ******************* Диагностика R21 ********************** /                          
##### До                    
R21#show ip bgp neighbors 10.0.18.1 advertised-routes                      
BGP table version is 15, local router ID is 10.250.21.250                             
Originating default network 0.0.0.0                     
     Network          Next Hop            Metric LocPrf Weight Path                            
 *>  10.128.16.0/20   10.0.18.10                             0 520 2042 i                   
 *>  10.250.18.250/32 10.0.18.10                             0 520 2042 i                    
 *>  10.250.21.250/32 0.0.0.0                  0         32768 i                         
 *>  10.250.22.250/32 10.0.18.6                0             0 101 i                     
 *>  10.250.23.250/32 10.0.18.10                             0 520 i                    
 *>  10.250.24.250/32 10.0.18.10               0             0 520 i                  
 *>  10.250.25.250/32 10.0.18.10                             0 520 i                
 *>  10.250.26.250/32 10.0.18.10                             0 520 i                   
Total number of prefixes 8                  
##### После                           
R21#show ip bgp neighbors 10.0.18.1 advertised-routes                    
BGP table version is 21, local router ID is 10.250.21.250                    
Originating default network 0.0.0.0                         
     Network          Next Hop            Metric LocPrf Weight Path                     
 *>  10.128.16.0/20   10.0.18.10                             0 520 2042 i                  
 *>  10.250.18.250/32 10.0.18.10                             0 520 2042 i                  
Total number of prefixes 2                 

###### Отдаем только сети Piter и маршрут по умолчанию.                      
/ ******************* end R21 ****************************** /                                
                           
                            
[Конфигурации устройств](./Conf/README.md)                   
                 





