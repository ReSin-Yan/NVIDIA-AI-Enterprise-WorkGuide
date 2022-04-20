Last update: 2022/4/20  

# NVIDIA-AI-Enterprise-WorkGuide  

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
| vSphere enterprise plus | 7.0.3 |  
| vCenter | 7.0.3 |  
| NVAIE | 1.1 |  
| NSX-T advanced loadbalance | 1.1 |  
| ESXi GPU Driver | 470.63 |  
| NVIDIA Liccense | 1.0.0 |  
| helm | >3.0 |  




**假設Tanzu已經安裝完成**  


**並且能正常配置帶有vGPU的VM(License sever 也配置完成)**  
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

10.登入到supvisorCluster的namespace  
```
kubectl-vsphere login --vsphere-username administrator@vsphere.local --server=x.x.x.x --insecure-skip-tls-verify 
```
11.切換到supvisorCluster的namespace  
```
kubectl config use-context namespace-NAME  
```
12. 建立TKC  

根據情況修改`name` `namespace` `vmClass` `storageClass` `defaultClass` 的名稱  
```
apiVersion: run.tanzu.vmware.com/v1alpha2
kind: TanzuKubernetesCluster
metadata:
   #cluster name
   name: tkgs-cluster-gpu-a100
   #target vsphere namespace
   namespace: demo
spec:
   topology:
     controlPlane:
       replicas: 1
       #storage class for control plane nodes
       #use `kubectl describe storageclasses`
       #to get available pvcs
       storageClass: nvaiestorage
       vmClass: best-effort-small
       #TKR NAME for Ubuntu ova supporting GPU
       tkr:
         reference:
           name: v1.20.8---vmware.1-tkg.2
     nodePools:
     - name: nodepool-a100-primary
       replicas: 2
       storageClass: nvaiestorage
       #custom VM class for vGPU
       vmClass: a1005g
       volumes:
           - name: containerd
             mountPath: /var/lib/containerd
             capacity:
               storage: 100Gi
       #TKR NAME for Ubuntu ova supporting GPU
       tkr:
         reference:
           name: v1.20.8---vmware.1-tkg.2
-vmware.1-tkg.2
   settings:
     storage:
       defaultClass: nvaiestorage
     network:
       cni:
        name: antrea
       services:
        cidrBlocks: ["198.51.100.0/12"]
       pods:
        cidrBlocks: ["192.0.2.0/16"]
       serviceDomain: managedcluster.local
```

```
kubectl apply -f createtkc.yaml
```

13.登入到TKC當中  
```
kubectl-vsphere login --vsphere-username administrator@vsphere.local --server=x.x.x.x --insecure-skip-tls-verify --tanzu-kubernetes-cluster-name tkgs-cluster-gpu-a100
```

14.切換到supvisorCluster的namespace  
```
kubectl config use-context tkgs-cluster-gpu-a100    
```

#### NVIDIA-AI-Enterprise NVIDIA Part  

[NVAIE GPU operator](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/install-gpu-operator-nvaie.html "link")  
NVIDIA 會使用一項叫做GPU-opeartor的技術  
將GPU驅動使用容器的模式來讓node使用  
vGPU本身的驅動程式是特別版本的  
所以需要`註冊好NVAIE`的帳號(在NGC上面)  
再利用帳號取得對應的docker images 以及helm chart(皆從NGC上面下載，並且需要註冊NVAIE才有)  

這邊假設沒有要設定smartNIC  

1.建立`gpu-operator`的命名空間  

```
kubectl create namespace gpu-operator
```

2.建利空的vGPU授權設定檔案  

```
sudo touch gridd.conf
```

3.將NLS下載下來的授權修改名稱為client_configuration_token.tok  

4.建立名為licensing-config的ConfigMap 

```
kubectl create configmap licensing-config -n gpu-operator --from-file=gridd.conf --from-file=<path>/client_configuration_token.tok
```

5.在gpu-operator這一個命名空間建立image pull secert(NGC帳號需要)   

```
export REGISTRY_SECRET_NAME=ngc-secret  
export PRIVATE_REGISTRY=nvcr.io/nvaie  
kubectl create secret docker-registry ${REGISTRY_SECRET_NAME} \
    --docker-server=${PRIVATE_REGISTRY} \
    --docker-username='$oauthtoken' \
    --docker-password='<password>' \
    --docker-email='<email-address>' \
    -n gpu-operator
```

6.新增NVAIE的helm Chart到連線端  

```
helm repo add nvaie https://helm.ngc.nvidia.com/nvaie \
  --username='$oauthtoken' --password='<password>' \
  && helm repo update
```

7.安裝NVIDIA GPU Operator  

```
helm install --wait gpu-operator nvaie/gpu-operator-1-1 -n gpu-operator
```

#### NVIDIA-AI-Enterprise Demo  

執行以下yaml檔案  
```
apiVersion: v1
kind: Pod
metadata:
  name: cuda-vectoradd
spec:
  restartPolicy: OnFailure
  containers:
  - name: cuda-vectoradd
    image: "nvidia/samples:vectoradd-cuda11.2.1"
    resources:
      limits:
         nvidia.com/gpu: 1
```

觀看執行logs  
```
kubectl logs cuda-vectoradd 
```




執行以下yaml檔案  
```
apiVersion: v1
kind: Service
metadata:
  name: jupyter
  labels:
    app: jupyter
spec:
  ports:
  - port: 80
    name: http
    targetPort: 8888
  selector:
    app: jupyter
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jupyter-app
  labels:
    app: jupyter
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jupyter
  template:
    metadata:
      labels:
        app: jupyter
    spec:
      containers:
      - name: jupyter
        image: nvcr.io/nvidia/tensorflow:22.01-tf2-py3
        command: ["/bin/sh"]
        args: ["-c","jupyter-lab --NotebookApp.token='' --ip=0.0.0.0 --port=8888 --allow-root"]
        ports:
        - containerPort: 8888
          protocol: TCP
        name: http
        resources:
          limits:
            nvidia.com/gpu: 1

```


### Install and Demo Tips   

由於A100本身只支援CUDA11版本之後  
所以在跑測試執行的時候需要使用CUDA 11版本之後來進行測試  
用CUDA10會整個環境卡死，GPU資源無法釋放  
