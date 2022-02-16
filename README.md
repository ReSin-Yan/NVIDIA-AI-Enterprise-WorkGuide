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
本文會著重於第二種方式來進行介紹  
圖片  

### 安裝步驟  

NVIDIA AI Enterprise 主要分成兩個部分  
兩個部份分別安裝，在安裝的時候也需要分頭進行安裝  

#### NVIDIA-AI-Enterprise Tanzu Part  

#### NVIDIA-AI-Enterprise NVIDIA Part  


### Tips   

由於A100本身只支援CUDA11版本之後  
所以在跑測試執行的時候需要使用CUDA 11版本之後來進行測試  
用CUDA10會整個環境卡死，GPU資源無法釋放  

