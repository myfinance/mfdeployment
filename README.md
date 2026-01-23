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

