1.Create an empty project.
oc new-project cotd-ab --display-name="COTD With A/B Deployment" --description=""COTD With A/B Deployment"

2.Create an application instance.
oc new-app --name='cotd1' -l name='cotd' php~https://github.com/leomachadorocha/cotd.git -e SELECTOR=cats

OBS: SELECTOR can be equal to cats, cities, or pets.

3.Create a second instance of the cotd application using a different SELECTOR.
oc new-app --name='cotd2' -l name='cotd' php~https://github.com/leomachadorocha/cotd.git -e SELECTOR=cities

4.Create a route that uses A/B routing across the two versions, initially using 50/50.
oc expose service cotd1 --name='ab-cotd-route'
oc set route-backends ab-cotd-route cotd1=50 cotd2=50

5.Test that your service returns images from each of the two categories you selected 50% of the time.
oc get routes (see <HOST/PORT>)
while true; do curl -s http://<HOST/PORT>/item.php | grep "data/images" | awk '{print $5}'; sleep 1; done
while true; do curl -s http://ab-cotd-route-lr-deployment.apps.gru.example.opentlc.com/item.php | grep "data/images" | awk '{print $5}'; sleep 1; done

6.
