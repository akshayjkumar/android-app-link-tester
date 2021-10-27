# Android App Links Tester


## What are App Links? 
App Links are regular web URLs that can take a user to a specific page/ location in the app. App Links can help to bypass the app selector dialog/ disambiguation dialog and navigate directly to the app improving the user experience. App Links are available on Android 6 and above.

> Android App Links use the Digital Asset Links API to establish trust that your app has been approved by the website to automatically open links for that domain. If the system successfully verifies that you own the URLs, the system automatically routes those URL intents to your app.

App Links which are not verified or properly associated to the website can result in selector dialog/ disambiguation dialog. Below Shell script is to test the app links registered/ associated with the app.

```
echo "App Links Tester\n"

# Exit code: zero for success and any non-zero for failure.
TEST_SUCCESS=0

# Wait time (in seconds) for the activity to resume.
WAIT_FOR_ACTIVITY=5

# Package id of the app to test.
PACKAGE_NAME="my.package.name"

# Domain name of the app.
DOMAIN_NAME="mydomain.com"

# Extract list of all subdomains of the app by filtering based on the above domain name. 
CMD_GET_DOMAINS="adb shell dumpsys package d | grep -e '$DOMAIN_NAME'"

# ADB shell command to force stop/ to close the app.  
CMD_ADB_KILL="adb shell am force-stop $PACKAGE_NAME"

#
# CMD_ADB_CHECK_FOREGROUND="adb shell dumpsys window windows | grep mCurrentFocus | cut -d'/' -f1 | rev | cut -d' ' -f1 | rev"
# mCurrentFocus will not work for Android 11 and above. Use mObscuringWindow.
#
CMD_ADB_CHECK_FOREGROUND="adb shell dumpsys window windows | grep -E 'mObscuringWindow' | cut -d '/' -f1 | sed 's/.* //g'"

#
# ABD command to open the app link/ deep link. 
# OPEN_APP_LINK="adb shell am start -a android.intent.action.VIEW -c android.intent.category.BROWSABLE -d"
# OPEN_APP_LINK="adb shell am start -a android.intent.action.VIEW -d"
#
OPEN_APP_LINK="adb shell am start -a android.intent.action.VIEW -d"

# Get the list of all subdomains.
DOMAIN_LIST=$(eval $CMD_GET_DOMAINS)

# DOMAIN_LIST is further split into an array of subdomains.
DOMAINS=($(echo $DOMAIN_LIST | grep -o '\b\S*\.com\b'))

for i in "${DOMAINS[@]}"
do
   CMD_ADB_INVOKE="$OPEN_APP_LINK 'https://$i/'"
   eval $CMD_ADB_INVOKE
   sleep $WAIT_FOR_ACTIVITY
   FOREGROUND_PACKAGE_NAME=$(eval $CMD_ADB_CHECK_FOREGROUND)
   echo "\nForeground task package name: \033[32m$FOREGROUND_PACKAGE_NAME\033[0m\n"
   if [ "$FOREGROUND_PACKAGE_NAME" == "$PACKAGE_NAME" ]
   then
    echo "✅ App Link working as expected for \033[32m $i \033[0m \n"
   else
    echo "❌ \033[31m App Link failed for $i \033[0m \n"
    TEST_SUCCESS=1
   fi 
   eval $CMD_ADB_KILL
done

exit $TEST_SUCCESS

```

## Test Success

<img width="1019" alt="app-links-verified" src="https://user-images.githubusercontent.com/16755620/139094385-fe3d5145-c33b-459f-83dc-bf5547b885a7.png">

## Test Failure

<img width="966" alt="app-links-failed" src="https://user-images.githubusercontent.com/16755620/139094669-09de44e1-25ac-4178-aaeb-2f8c05ce293b.png">

