oc project $DEVEL_PROJ_NAME

IS_NAME=`oc get is| tail -1|awk '{print $1}'`

#get full name of the image
FULL_IMAGE_NAME=`oc describe is ${IS_NAME} | grep -a1 "Tag" | tail -1 | awk '{print $6}'`

#Tag to promote to QA
oc tag $FULL_IMAGE_NAME $DEVEL_PROJ_NAME/${IS_NAME}:promote


#This should automatically initiate deployment

oc project $QA_PROJ_NAME

#Find the DeploymentConfig to see if this is a new deployment or just needs an update
DC_ID=`oc get dc | tail -1| awk '{print $1}'`

if [ $DC_ID == "NAME" ]; then
	oc new-app $DEVEL_PROJ_NAME/${IS_NAME}:promote --name=$APP_NAME
	SVC_ID=`oc get svc | tail -1 | awk '{print $1}'`
	oc expose service $SVC_ID --hostname=$APP_HOSTNAME
fi


#find the new rc based on the FULL_IMAGE_NAME=$FULL_IMAGE_NAME
RC_ID=""
attempts=75
count=0
while [ -z "$RC_ID" -a $count -lt $attempts ]; do
	RC_ID=`oc get rc | grep $FULL_IMAGE_NAME | awk '{print $1}'`
	count=$(($count+1))
	sleep 5
done

if [ -z "$RC_ID" ]; then
  echo "Fail: App deployment was not successful"
  exit 1 
fi

#Scale the app to 1 pod (just to make sure)
scale_result=`oc scale rc $RC_ID --replicas=1| awk '{print $3}'`

if [ $scale_result != "scaled" ]; then
  echo "Fail: Scaling not successful"
  exit 1 
fi
