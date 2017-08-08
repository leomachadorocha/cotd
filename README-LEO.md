# Create the Projects # 

1. Create an empty project.
```
oc new-project cotd-ab --display-name="COTD With A/B Deployment" --description="COTD With A/B Deployment"
```

2. Create an application instance, for serving cats.
```
oc new-app --name='cotd1' -l name='cotd' php~https://github.com/leomachadorocha/cotd.git -e SELECTOR=cats
```

OBS: SELECTOR can be equal to cats, cities, or pets.

3. Create a second instance of the cotd application, for serving cities, using a different SELECTOR.
```
oc new-app --name='cotd2' -l name='cotd' php~https://github.com/leomachadorocha/cotd.git -e SELECTOR=cities
```


# A/B Deployment # 

4. Create a route that uses A/B routing across the two versions, initially using 50/50.
```
oc expose service cotd1 --name='ab-cotd-route'
oc set route-backends ab-cotd-route cotd1=50 cotd2=50
```

5. Test that your service returns images from each of the two categories you selected.

OBS: Servers use cookies to remember state between calls. The A/B router uses affinity to route subsequent requests to the same back-end instance of the application. Because browsers send cookies when issuing HTTP requests, during testing you may end up reaching the same back-end instance for each request. If using a browser, you can clear the cookies, use incognito mode, or use different browsers to retrieve different pictures. We can use curl to avoid this issue.
```
oc get routes (see <HOST/PORT>)
```
```
while true; do curl -s http://<HOST/PORT>/item.php | grep "data/images" | awk '{print $5}'; sleep 1; done
while true; do curl -s http://ab-cotd-route-lr-deployment.apps.gru.example.opentlc.com/item.php | grep "data/images" | awk '{print $5}'; sleep 1; done
```

6. Increase the weights for one service by 20% and verify that you retrieve seven images of one type and three of the other for every ten calls to the route. 
```
oc edit route ab-cotd-route
```

7. Test again.
```
while true; do curl -s http://ab-cotd-route-lr-deployment.apps.gru.example.opentlc.com/item.php | grep "data/images" | awk '{print $5}'; sleep 1; done

```


# BLUE/GREEN Deployment #

1. Delete the previous route.
```
oc delete route ab-cotd-route
```

2. Create new route, serving traffic to the BLUE application (cotd1).
```
oc expose svc/cotd1 --name=bluegreen-cotd-route
```

3. Test.
```
while true; do curl -s http://bluegreen-cotd-route-lr-deployment.apps.gru.example.opentlc.com/item.php | grep "data/images" | awk '{print $5}'; sleep 1; done
```

4. While the curl commnad is running, open new terminal and change the traffic to the GREEN application (cotd2).
```
oc patch route/bluegreen-cotd-route -p '{"spec":{"to":{"name":"cotd2"}}}'
```

5. See, in the first terminal, that now the traffic has been redirected to cotd2 application.  


