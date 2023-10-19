1. _Criar e gerenciar um statefulset:_
  - _Criar um statefulset simples com um par de pods_

    **Solução:**
    - Como estou rodando um cluster local (em VMs), eu não tenho autoprovisionamento de volumes disponível, e para que o statefulset tenha o PVC atendido quando criado é necessário criar os PVs antes. Para isso criei os arquivos `redis-pv-{0-3}.yml`.
    ```
    ubuntu@k8s-01:~/PICK-linuxtips/descomplicando-kubernetes/day-7/licao-de-casa$ k apply -f redis-pv-0.yml
    ubuntu@k8s-01:~/PICK-linuxtips/descomplicando-kubernetes/day-7/licao-de-casa$ k apply -f redis-pv-1.yml
    ubuntu@k8s-01:~/PICK-linuxtips/descomplicando-kubernetes/day-7/licao-de-casa$ k apply -f redis-pv-2.yml

    ubuntu@k8s-01:~/PICK-linuxtips/descomplicando-kubernetes/day-7/licao-de-casa$ k get pv
    NAME           CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                        STORAGECLASS   REASON   AGE
    redis-data-0   1Gi        RWO            Retain           Bound       default/redis-data-redis-0   giropops                34m
    redis-data-1   1Gi        RWO            Retain           Bound       default/redis-data-redis-1   giropops                34m
    redis-data-2   1Gi        RWO            Retain           Available                                giropops                34m

    ```

    ```
    ubuntu@k8s-01:~/PICK-linuxtips/descomplicando-kubernetes/day-7/licao-de-casa$ k apply -f  redis-statefulset.yml
    ubuntu@k8s-01:~/PICK-linuxtips/descomplicando-kubernetes/day-7/licao-de-casa$ k get pods
    NAME                               READY   STATUS    RESTARTS   AGE
    redis-0                            1/1     Running   0          29m
    redis-1                            1/1     Running   0          29m
    ```
  - _Escalar para mais pods_

    **Solução**
    ```
    ubuntu@k8s-01:~/PICK-linuxtips/descomplicando-kubernetes/day-7/licao-de-casa$ k scale statefulset redis  --replicas=3
    statefulset.apps/redis scaled

    ubuntu@k8s-01:~/PICK-linuxtips/descomplicando-kubernetes/day-7/licao-de-casa$ k get pods
    NAME                               READY   STATUS    RESTARTS   AGE
    redis-0                            1/1     Running   0          36m
    redis-1                            1/1     Running   0          36m
    redis-2                            1/1     Running   0          3s

    
    ubuntu@k8s-01:~/PICK-linuxtips/descomplicando-kubernetes/day-7/licao-de-casa$ k get pv
    NAME           CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                        STORAGECLASS   REASON   AGE
    redis-data-0   1Gi        RWO            Retain           Bound    default/redis-data-redis-0   giropops                38m
    redis-data-1   1Gi        RWO            Retain           Bound    default/redis-data-redis-1   giropops                38m
    redis-data-2   1Gi        RWO            Retain           Bound    default/redis-data-redis-2   giropops                38m
    
    ubuntu@k8s-01:~/PICK-linuxtips/descomplicando-kubernetes/day-7/licao-de-casa$ k get pvc
    NAME                 STATUS   VOLUME         CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    redis-data-redis-0   Bound    redis-data-0   1Gi        RWO            giropops       37m
    redis-data-redis-1   Bound    redis-data-1   1Gi        RWO            giropops       37m
    redis-data-redis-2   Bound    redis-data-2   1Gi        RWO            giropops       55s
    ```
  - _Apagar um pode e validar comportamento_

    **Solução**
    ```
    ubuntu@k8s-01:~/PICK-linuxtips/descomplicando-kubernetes/day-7/licao-de-casa$ k delete pod redis-0
    pod "redis-0" deleted

    ubuntu@k8s-01:~/PICK-linuxtips/descomplicando-kubernetes/day-7/licao-de-casa$ k get pods -w
    NAME                               READY   STATUS    RESTARTS   AGE
    redis-0                            1/1     Running   0          39s
    redis-1                            1/1     Running   0          39m
    redis-2                            1/1     Running   0          3m2s
    ```

2. _Expor o statefulset com um service do tipo ClusterIP_

   **Solução**

   Aqui eu mudei um pouco o que o exercicio pedia, invés de expor o statefulset num serviço tipo ClusterIP, eu optei por um serviço headless (que teoricamente também é do tipo ClusterIP), e subi um deployment da aplicação giropops-senhas apontando para o serviço do redis. Finalmente, expus internamente o giropops senhas com um service do tipo ClusterIP e externamente com um service do tipo NodePort:

    ```
    ubuntu@k8s-01:~/PICK-linuxtips/descomplicando-kubernetes/day-7/licao-de-casa$ k apply -f redis-headless-svc.yml
    ubuntu@k8s-01:~/PICK-linuxtips/descomplicando-kubernetes/day-7/licao-de-casa$ k get svc
    NAME                       TYPE        CLUSTER-IP      EXTERNAL-IP                  PORT(S)          AGE
    kubernetes                 ClusterIP   10.96.0.1       <none>                       443/TCP          18d
    redis-headless-svc         ClusterIP   None            <none>                       6379/TCP         47m

    ubuntu@k8s-01:~/PICK-linuxtips/descomplicando-kubernetes/day-7/licao-de-casa$ k apply -f giropops-senhas-deployment.yml
    ubuntu@k8s-01:~/PICK-linuxtips/descomplicando-kubernetes/day-7/licao-de-casa$ k get deploy
    NAME              READY   UP-TO-DATE   AVAILABLE   AGE
    giropops-senhas   2/2     2            2           48m

    ubuntu@k8s-01:~/PICK-linuxtips/descomplicando-kubernetes/day-7/licao-de-casa$ k apply -f giropops-senhas-clusterip-svc.yml

    ubuntu@k8s-01:~/PICK-linuxtips/descomplicando-kubernetes/day-7/licao-de-casa$ k get svc
    NAME                       TYPE        CLUSTER-IP      EXTERNAL-IP                  PORT(S)          AGE
    giropops-senhas            ClusterIP   10.98.158.190   <none>                       8888/TCP         46m
    kubernetes                 ClusterIP   10.96.0.1       <none>                       443/TCP          18d

    ubuntu@k8s-01:~/PICK-linuxtips/descomplicando-kubernetes/day-7/licao-de-casa$ curl 10.98.158.190:8888
    <!DOCTYPE html>
    <html lang="pt-BR">

    <head>
    <meta charset="UTF-8" />
    ....

    ubuntu@k8s-01:~/PICK-linuxtips/descomplicando-kubernetes/day-7/licao-de-casa$ k apply -f giropops-senhas-nodeport-svc.ym

    ubuntu@k8s-01:~/PICK-linuxtips/descomplicando-kubernetes/day-7/licao-de-casa$ k get svc
    NAME                       TYPE        CLUSTER-IP      EXTERNAL-IP                  PORT(S)          AGE
    giropops-senhas            ClusterIP   10.98.158.190   <none>                       8888/TCP         52m
    giropops-senhas-nodeport   NodePort    10.102.10.38    100.25.3.231,54.236.54.171   8888:32288/TCP   45m
    kubernetes                 ClusterIP   10.96.0.1       <none>                       443/TCP          18d
    redis-headless-svc         ClusterIP   None            <none>                       6379/TCP         53m

    ```
3. _Criar um service do tipo ExternalName_

   **Solução**

   ```
    ubuntu@k8s-01:~/PICK-linuxtips/descomplicando-kubernetes/day-7/licao-de-casa$ k apply -f external-name-svc.yml

    ubuntu@k8s-01:~/PICK-linuxtips/descomplicando-kubernetes/day-7/licao-de-casa$ k get svc
    NAME                       TYPE           CLUSTER-IP      EXTERNAL-IP                  PORT(S)          AGE
    external-name-service      ExternalName   <none>          www.google.com               <none>           11m
    giropops-senhas            ClusterIP      10.98.158.190   <none>                       8888/TCP         81m
    giropops-senhas-nodeport   NodePort       10.102.10.38    100.25.3.231,54.236.54.171   8888:32288/TCP   74m
    kubernetes                 ClusterIP      10.96.0.1       <none>                       443/TCP          18d
    redis-headless-svc         ClusterIP      None            <none>                       6379/TCP         82m

   ```
   Dessa vez a validação falhou, subi um busybox pra dar um wget pro serviço e algum problema de certificado atrapalhou a execução, mas o dns foi resolvido.
