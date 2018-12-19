## Check disk volume
lsblk

df -h

## Extend disk volume
growpart /dev/xvda1

## Resize disk volume
resize2fs /dev/xvda1
