---
layout: post
title: Xcode自动更新build号
# background-image: 
date: 2020-05-09
published: true
category: Script
tags:
        - xcode
---

```shell
plist=${INFOPLIST_FILE}
buildnum=$(/usr/libexec/PlistBuddy -c "Print CFBundleVersion" "${plist}")

if [[ "${buildnum}" == "" ]]; then
echo " Error：No Build Number In Plist"
exit 2
fi

if [ $CONFIGURATION == Release ]; then
echo " Release"
buildnum=$(date +%Y%m%d%H%M%S)
buildnum="$buildnum.release"
/usr/libexec/PlistBuddy -c "Set CFBundleVersion $buildnum" "${plist}"
echo " CFBundleVersion----->>> $buildnum"
else
echo $CONFIGURATION " Debug"
buildnum=$(date +%Y%m%d%H%M%S)
buildnum="$buildnum.debug"
/usr/libexec/PlistBuddy -c "Set CFBundleVersion $buildnum" "${plist}"
echo " CFBundleVersion----->>> $buildnum"
fi
```