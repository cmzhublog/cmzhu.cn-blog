## 使用shell 脚本备份

使用shell 脚本备份命名空间下的所有资源

执行方式， 备份xxx命名空间下的所有服务

```bash
$ bash ./dump-k8s-yaml.sh ./cmzhu /root/.kube/config cmzhu
```



shell 脚本`dump-k8s-yaml.sh`

```bash
#!/bin/bash

useage(){
    echo "Useage:"
    echo "  bash ./dump-k8s-yaml.sh DUMPDIR KUBECONFIG [NAMESPACE]"
}

if [ $# -lt 1 ];then
    useage
    exit
fi

dumpDir=$1
KF=$2
NS=$3

resourceList=(
  #componentstatuses       # cs
  configmaps              # cm
  secrets
  persistentvolumeclaims  # pvc
  events                  # ev
  serviceaccounts         # sa
  endpoints               # ep
  services                # svc
  ingress
  daemonsets              # ds
  deployments             # deploy
  replicasets             # rs
  statefulsets            # sts
  jobs
  cronjobs                # cj
  pods                    # po
)

showResourceList(){
  kubectl --kubeconfig=$KF -n=$NS get nodes
  kubectl --kubeconfig=$KF --sort-by=.metadata.name -n=$NS get pods
  kubectl --kubeconfig=$KF -n=$NS get cm,secrets,pvc,ev,sa,ep,svc,ingress,ds,deploy,rs,sts,jobs,cj
}

printList(){
  for aa in ${resourceList[@]};
  do
    aList=$(kubectl --kubeconfig=$KF -n=$NS get $aa | grep -v NAME | awk '{print $1}')
    if [ ! "${aList[*]}"x == "x" ];then
      [ -d $dumpDir/$aa ] || mkdir -p $dumpDir/$aa
      for i in $aList;
      do
        echo $aa $i
        kubectl --kubeconfig=$KF -n=$NS get $aa $i -o yaml > $dumpDir/$aa/$i.yaml
      done
    fi
  done
}

zipExec(){
  #sudo apt install zip unzip
  #zip -v
  #unzip -v
  zip -r -q $dumpDir.zip file $dumpDir
  rm -rf $dumpDir
}

# create namespaces yaml
if [ ! -d $dumpDir ];then
  mkdir -p -m 777 ./$dumpDir
fi

kubectl --kubeconfig=$KF get namespaces $NS -o yaml > $dumpDir/$NS.yaml

# create pv yaml
pvList=$(kubectl --kubeconfig=$KF get pv | grep "$NS/" | awk '{print $1}')
if [ ! "${pvList[*]}"x == "x" ];then
  [ -d $dumpDir/persistentvolumes ] || mkdir -p $dumpDir/persistentvolumes
  for i in ${pvList[@]}
  do
    echo persistentvolumes $i
    kubectl --kubeconfig=$KF get pv $i -o yaml > $dumpDir/persistentvolumes/$i.yaml
  done
fi

echo "----[showResourceList]-----------------------"
showResourceList
echo "----[printList]-----------------------"
printList
echo "----[zipExec]-----------------------"
zipExec
echo "export ${NS} ymal completed!"
```

