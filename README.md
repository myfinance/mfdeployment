# mfdeployment

## value files

- the values_base.yaml contains  properties which are allways the same no matter the version or the environment
- the value files in the env directory contains properties which are spezific for each environment like external ports.
this files will never be stages but have to changed directly
- the value files in the versions directory change often and depend on the software version like the image-version 

## staging

just copy the versions_dev.yaml to the versions_prod.yaml and commit

## backup

process automaticly. You can change the schedule in the evironment properties dumpschedule

## data restore

kubectl exec -it <mongo-podname> -n <namespace> -- /bin/sh
mongorestore --username=root --password=<pw> --authenticationDatabase=admin --drop --gzip --archive=/backup/mfbackup_<timestamp>.gz

or from your local maschine:
# 1. Copy the dump into the pod
kubectl --kubeconfig ~/.kube/config-k3s cp ~/repos/dump/mfbackup_2026-09-06_22-01-01.gz \
  mfprod/$(kubectl --kubeconfig ~/.kube/config-k3s get pod -n mfprod -l app=mfmongo -o jsonpath='{.items[0].metadata.name}'):/tmp/mfbackup.gz
# 2. Run the restore locally inside the pod
kubectl --kubeconfig ~/.kube/config-k3s exec -it -n mfprod deploy/mfmongo -- \
  mongorestore --username=root --password=vulkan --authenticationDatabase=admin --drop --gzip --archive=/tmp/mfbackup.gz
# 3. Clean up the temporary file in the pod
kubectl --kubeconfig ~/.kube/config-k3s exec -n mfprod deploy/mfmongo -- rm /tmp/mfbackup.gz

Attention: you have to restart all myfinance pods:
When mongorestore ran with --drop, it restored database collections and authentication records (restoring users from "archive /tmp/mfbackup.gz").
MongoDB immediately invalidates all active sessions on existing TCP connections when users and authentication metadata are dropped or restored. Because the microservices (mftransactions, mfmarketdata, etc.) were already running and held open sockets in their connection pools from before the restore, they reused those sockets without re-authenticating, resulting in: Command failed with error 13 (Unauthorized): 'Command find requires authentication'.

