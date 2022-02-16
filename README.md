Last update: 2022/2/15  

# NVIDIA-AI-Enterprise-WorkGuide  

### NVIDIA-AI-Enterprise簡易介紹     

[NVIDIA AI ENTERPRISE](https://www.nvidia.com/zh-tw/data-center/products/ai-enterprise-suite/ "link")  
NVIDIA AI Enterprise 是提供人工智慧與資料分析軟體的端對端、雲端原生套件，經由 NVIDIA 最佳化、認證和支援，可在使用 NVIDIA 認證系統™的 VMware vSphere 上執行；內含 NVIDIA 的關鍵致能技術，能在現代混合雲快速部署、管理及擴充人工智慧工作負載。  

如何實現分為兩種方式  

第一種為傳統虛擬機模式  
透過vGPU的方式分配卡片給VM，再從VM端安裝docker的方式來執行  
跟舊有的vGPU方式一樣，差別在於多了容器端的應用支援(NVIDIA GPU Cloud Support)  
圖片  

第二種為容器模式  
透過Tanzu來進行底層環境的部屬，搭配GPU來讓建立出來的環境都有GPU的功能  
搭配版本為vSphere with Tanzu 而非 Tanzu Kubernetes Grid  
本文會著重於第二種方式來進行介紹  
圖片  

### 安裝步驟  

NVIDIA AI Enterprise 主要分成兩個部分  
兩個部份分別安裝，在安裝的時候也需要分頭進行安裝  

硬體配置  
 | 名稱 | 角色  |
|-------|-------|
| 伺服器 | PowerEdge R740 |  
| 卡片 | NVIDIA A100 |  
| CPU | Intel(R) Xeon(R) Silver 4114 CPU @ 2.20GHz |  
| RAM | 256GB |  
| DISK | Total 1TB wihh Raid 0 |  
| 網卡 | 最少兩張(需要兩個網段) |  

**以上僅為測試環境規格(只有一台伺服器)，在建立過程中因為效能問題重建多次，所以建議依照正確的Tanzu環境建議來進行配置**  


軟體配置  
 | 名稱 | 角色  |
|-------|-------|
| vSphere | 7.0.3 |  
| vCenter | 7.0.3 |  
| NVAIE | 1.1 |  
| NSX-T advanced loadbalance | 1.1 |  
| ESXi GPU Driver | 470.63 |  
| NVIDIA Liccense | 1.0.0 |  




**假設Tanzu已經安裝完成**  


**並且能正常配置帶有vGPU的VM**  
[vGPU quick start guide](https://docs.nvidia.com/grid/latest/grid-software-quick-start-guide/index.html "link")  

**接下來設定會著重在環境設定**  

#### NVIDIA-AI-Enterprise Tanzu Part  

[設定步驟參考文件](https://docs.nvidia.com/ai-enterprise/deployment-guide/dg-vsphere-tanzu.html "link")  
參考到設定好虛擬機器類別就好  
後面設定包含yaml都是錯誤的(會需要參考到另外一篇文章進行設定)  

Tanzu的版本是vSphere with Tanzu 而非 Kubernetes with Tanzu  
兩者是不一樣的架構  
目前只支援vSphere with Tanzu  

Tanzu部分需要設定的地方不多  
主要著重在`vmClass`的設定  
相較於舊版7.0.2  
7.0.3新增了一條`PCIE裝置`  
透過這種方式新增一種vmClass  
讓在建立的時候配置好  

以下是參考流程  

1.進入已經起好vsphere with 的環境中，點選`工作負載管理`  
![img](https://github.com/ReSin-Yan/NVIDIA-AI-Enterprise-WorkGuide/blob/main/img/1.PNG)  

2.點選`服務` > `虛擬機器服務` > `管理`  
![img](https://github.com/ReSin-Yan/NVIDIA-AI-Enterprise-WorkGuide/blob/main/img/2.PNG)  

3.點選`虛擬機器類別` > `建立虛擬機器類別`  
![img](https://github.com/ReSin-Yan/NVIDIA-AI-Enterprise-WorkGuide/blob/main/img/3.PNG)  

4.設定包含`名稱``vCPU數量``CPU資源保留``記憶體大小` PCI裝置選擇`是` 開啟PCI裝置功能會自動將記憶體保留變成100趴   
![img](https://github.com/ReSin-Yan/NVIDIA-AI-Enterprise-WorkGuide/blob/main/img/4.PNG)  

5.點選`新增PCI裝置` > `NVIDIA vGPU`  如果環境內包含網路卡SmartNIC則額外再選擇動態DirectPath IO    
![img](https://github.com/ReSin-Yan/NVIDIA-AI-Enterprise-WorkGuide/blob/main/img/5.PNG)  

6.選擇`卡片型號`  `GPU共用` `GPU模式` `GPU記憶體` `vGPU數目`  
其中卡片型號為目前可認得到的卡片(有裝GPU Driver並且有配置好基本設定)  
GPU共用分為兩種，傳統vGPU方式跟MIG模式，如果一開始在底層配置好了MIG則選擇`多執行個體GPU共用`，如果是沒有配置MIG則選擇`時間共用`  
GPU記憶體測試起來感覺有BUG，沒辦法設定10 20 40的數字(Demo環境設定5)  
![img](https://github.com/ReSin-Yan/NVIDIA-AI-Enterprise-WorkGuide/blob/main/img/6.PNG)  

7.點選`確認`    
![img](https://github.com/ReSin-Yan/NVIDIA-AI-Enterprise-WorkGuide/blob/main/img/7.PNG)  

8.進到命名空間配置虛擬機器服務，`虛擬機器服務`> `管理虛擬機器類別`   
![img](https://github.com/ReSin-Yan/NVIDIA-AI-Enterprise-WorkGuide/blob/main/img/8.PNG)  

9.將剛剛建立的類別新增至此命名空間   
![img](https://github.com/ReSin-Yan/NVIDIA-AI-Enterprise-WorkGuide/blob/main/img/9.PNG)  



#### NVIDIA-AI-Enterprise NVIDIA Part  


### Tips   

由於A100本身只支援CUDA11版本之後  
所以在跑測試執行的時候需要使用CUDA 11版本之後來進行測試  
用CUDA10會整個環境卡死，GPU資源無法釋放  

