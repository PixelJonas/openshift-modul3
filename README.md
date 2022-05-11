# OpenShift Workshop - Modul 3

## Vorbereitung

Nehmt euren User `userX` und loggt euch im CodeReadyWorkspaces ein.

> Link kommt im Chat

Default Credentials:

Username:
`userX` (z.B. `user25`)
Password:
`r3dh4t1!`

Klick auf "Add Workspace" und dann öffne den Tab "Custom Workspace".
Kopiere folgende URL in das Textfeld (neben "Select a devfile template") und klick auf "Load Devfile"

```
https://raw.githubusercontent.com/nikolaus-lemberski/openshift-modul3/main/cluster-preperation/workshop-tools/workshop-devfile.yaml
```

Scroll nach unten und klick auf "Create & Open". Das erstellen der Workspaces wird jetzt ein wenig dauern (da alle gleichzeitig starten).

Wenn der Workspace startet, bekommt ihr unten rechts 3 Popups

> Do you trust the authors of https://github.com/nikolaus-lemberski/openshift-modul3.git ?

klickt hier auf `Yes, I trust`

> Do you trust the authors of https://github.com/quarkusio/quarkus-quickstarts.git ?

klickt hier auf `Yes, I trust`

> Do you want to install the recommended extensions redhat/java,vscode/typescript-language-features for your workspace ?

klickt hier auf `No`


In dem Workspace findet ihr ein paar Beispielprojekte und ein paar "Tooling" Container. 
Die Tooling Container findet ihr rechts, wenn ihr auf das "Box" Icon klickt (Rechte Tooling Leiste, drittes Icon von oben)

Öffnet `openshift-tools` und klickt auf `New terminal`. Wählt als `working dir` `modul3` aus.

Es öffnet sich jetzt ein neues Terminal, in welchem ihr euch noch via `oc` CLI anmelden könnt

> ersetzt `userX` durch euren Benutzernamen

`oc login --insecure-skip-tls-verify https://${KUBERNETES_SERVICE_HOST} -u userX -p r3dh4t1!`

Nach der Eingabe des Passwortes sind wir erfolgreich eingeloggt und erhalten eine Willkommens-Nachricht von OpenShift.
Username und Adresse werden vom Trainer für jeden Teilnehmer zur Verfügung gestellt. Anschließend loggen wir uns noch in die Web Konsole von OpenShift ein (Adresse wird ebenfalls vom Trainer zur Verfügung gestellt).

## 1 - App Deployment

> **⚠ HINWEIS:**  
> Bitte alle Aufgaben selbst lösen und dann über die [Lösung](solutions/solution-1/) prüfen, ob alles richtig gemacht wurde.

### Projekt erstellen

Alle Applikationen in OpenShift werden in Projekten organisiert. In einem Projekt können viele Applikationen enthalten sein, sie befinden sich im gleichen _namespace_ und können miteinander über Services kommunizieren.

Als erstes erstellen wir ein Projekt **hello** über die OpenShift CLI. Damit wir mit den Projektnamen nicht durcheinanderkommen, stellt jeder vor den Projektnamen seinen Usernamen, also z.B. **user123-hello**.

### Applikation bauen und deployen

In unserem neuen Projekt deployen wir eine fertige nodejs Anwendung über OpenShift Source-to-Image (s2i). Über `oc new-app -h` kann die Hilfe aufgerufen werden, wie eine Applikation aus einem Git repository von OpenShift gebaut und deployed werden kann.

Die Applikation:
* liegt in Git unter folgender Adresse:  
https://github.com/nikolaus-lemberski/openshift-modul3
* dort im Unterordner (context directory) _projects/project-1_
* erhält als Applikationsnamen "hello"
* nutzt nodejs in der Version 16 mit dem ubi8 baseimage, als ImageStream _nodejs:16-ubi8-minimal_ in OpenShift bereitgestellt
Hinweis: es muss kein Dockerfile erstellt werden!
* als build strategy soll _source_ verwendet werden

Über `oc get all` kann alles, was der `oc new-app` command erstellt hat, angesehen werden. Es werden zwei pods gebaut, erst ein "build" pods der die Anwendung baut, danach der pod mit der Anwendung.

**Zusatzaufgaben:** 

* Den Build Log verfolgen
* Eine shell im container der app öffnen und mit curl den app root öffnen
* Die Applikation in der Web Konsole, Developer Perspektive untersuchen

### Health Checks

Health Checks dienen in Kubernetes dazu, dass Kubernetes den Status einer Anwendung überprüfen kann und sollte der Health Check fehlschlagen, die Anwendung neu startet. Health Checks können bei der Erstellung einer Anwendung über das Deployment oder die DeploymentConfig konfiguriert werden. Bei einer bestehenden Anwendung wird der Health Check am einfachsten über die Web Konsole konfiguriert.

Die Anwendung stellt zwei Health Checks bereit:

* `/health/readiness`  
Der readiness check, der - sobald erfolgreich - an OpenShift das Signal gibt, dass nun traffic an die Anwendung geroutet werden darf.
* `/health/liveness`  
Der liveness check, der von OpenShift kontinuierlich überprüft wird. Schlägt die liveness probe fehl, wird der pod mit der Anwendung automatisch gestoppt. Da dann weniger pods aktiv sind als im ReplicaSet vorgebeben, wird automatisch ein neuer pod mit der Anwendung erstellt.

**Aufgabe:** Konfiguration von readiness und liveness Health Checks über die Web Konsole.

### Applikation öffentlich aufrufbar machen

Zuletzt erstellen wir eine _route_ für die app, um diese öffentlich aufrufbar zu machen, und testen (z.B. mit curl), ob die Anwendung erreichbar ist.

### Projekt löschen

Um Ressourcen für weitere Projekte freizugeben, löschen wir das Projekt wieder.


## 2 - App Deployment mit Fehlersuche

> **⚠ HINWEIS:**  
> Bitte alle Aufgaben selbst lösen und dann über die [Lösung](solutions/solution-2/) prüfen, ob alles richtig gemacht wurde.

## Projekt erstellen

Wie gehabt erstellen wir wieder ein Projekt, diesmal mit dem Namen **nodeapp** und vorangestelltem Usernamen, also nach dem Schema  **user123-nodeapp**.

Zuerst speichern wir das folgende Deploymentfile als "Deployment.yml" im Ordner `modul3`:

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodeapp
  labels:
    app: nodeapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nodeapp
  template:
    metadata:
      labels:
        app: nodeapp
    spec:
      containers:
      - name: nodeapp
        image: quay.io/nlembers/project-2:v1.0
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: nodeapp
spec:
  selector:
    app: nodeapp
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
```

Anschließend deployen wir die Applikation in unserem neu erstellten Projekt:  
`oc create -f Deployment.yml`

## Fehlersuche und -behebung

Nachdem die app erstellt ist, wird der pod mit der app crashen. Die Aufgabe ist nun, die Ursache für den crash zu finden und das Problem zu beheben.

### Applikation öffentlich aufrufbar machen

Nachdem das Problem erfolgreich behoben wurde, erstellen wir eine _route_ für die app, um diese öffentlich aufrufbar zu machen, und testen (z.B. mit curl), ob die Anwendung erreichbar ist.

### Projekt löschen

Um Ressourcen für weitere Projekte freizugeben, löschen wir das Projekt wieder.


## 3 - Helm

> **⚠ HINWEIS:**  
> Bitte alle Aufgaben selbst lösen und dann über die [Lösung](solutions/solution-3/) prüfen, ob alles richtig gemacht wurde.

## Projekt erstellen

Wie gehabt erstellen wir wieder ein Projekt, diesmal mit dem Namen **helmapp** und vorangestelltem Usernamen, also nach dem Schema  **user123-helmapp**.

## Helm installieren

Helm ist ein Paketmanager für Kubernetes und erlaubt - neben dem Deployment fertiger Anwendungen als helm chart - die Erstellung eigener Anwendungslandschaften. Dazu installieren wir Helm auf unserem Rechner, falls noch nicht vorhanden:

[Helm](https://helm.sh/docs/intro/install/)

Wer zum ersten mal mit Helm zu tun hat, kann also Vorbereitung den [Helm Chart Template Guide](https://helm.sh/docs/chart_template_guide/getting_started/) durcharbeiten. 

## Helm Chart erstellen

Wir erstellen ein neues Helm Chart mit dem Namen **tasks**:  
`helm create tasks`

Und wechseln in das neu erstellte Verzeichnis _tasks_.

Unsere app besteht aus einer Java Spring Anwendung, die eine REST Schnittstelle anbietet und Daten in einer Datenbank (MariaDB) speichert. Wir möchten die Java Anwendung mit der Datenbank zusammen in unserem Helm Chart konfigurieren und über Helm deployen.

### MariaDB hinzufügen

In _Chart.yaml_ fügen wir MariaDB als _dependency_ hinzu. 

* Name: mariadb
* Version: 10.7.3
* Repository: https://charts.bitnami.com/bitnami

In _values.yaml_ konfigurieren wir die MariaDB:

```yml
mariadb:
  auth:
    username: tasksuser
    password: supersecretpwd
    database: tasksdb
  primary:
    podSecurityContext:
      enabled: false
    containerSecurityContext:
      enabled: false
```

Damit die dependency hinzugefügt wird, weisen wir Helm zum Aktualisieren der Abhängigkeiten an:  
`helm dependency update`

### Java Anwendung hinzufügen

Ebenfalls in _values.yaml_ wird das _image_ für unsere Java Anwendung konfiguriert:

* repository: quay.io/nlembers/spring-tasks
* tag: v1.1
* pullPolicy: bitte eine geeignete pull policy angeben

Die Spring Anwendung benötigt die Zugangsdaten für die Datenbank sowie den Treibernamen. Diese müssen als environment Variablen übergeben werden. Environment Variablen werden in _values.yaml_ hinzugefügt und müssen dann noch im Deployment (_/templates/deployment.yaml_) über das Helm templating eingelesen werden.

Die Spring Anwendung erwartet die folgenden Umgebungsvariablen:

* `SPRING_DATASOURCE_DRIVER_CLASS_NAME`  
`org.mariadb.jdbc.Driver`
* `SPRING_DATASOURCE_URL`  
`jdbc:mariadb://tasks-mariadb:3306/tasksdb`
* `SPRING_DATASOURCE_USERNAME`  
der Username (sh. oben)
* `SPRING_DATASOURCE_PASSWORD`
das Passwort (sh. oben)

### Anwendung installieren

In unserem Verzeichnis _tasks_ nutzen wir Helm für die Installation:  
`helm install tasks .``

### Anwendung testen

Wie in den vorherigen Übungen können wir nun über `oc get all`, `oc logs` etc. die Anwendung prüfen. Läuft alles korrekt, erstellen wir eine Route auf den Service der Java Anwendung und rufen diese im Browser unter dem Pfad *'/api'* auf. Es öffnet sich ein HAL Explorer, mit dem man mit dem REST Service interagieren kann. Curl oder httpie etc. können natürlich ebenso verwendet werden.

**Zusatzaufgaben:** 

* Erstellen von ein paar Tasks über den HAL Explorer oder curl
* Eine shell in die MariaDB öffnen
* In der MariaDB die Tabelle in der tasksdb auf das Vorhandensein der erstellten Tasks prüfen

### Projekt löschen

Um Ressourcen für weitere Projekte freizugeben, löschen wir das Projekt wieder.


## 4 - Service Mesh

> **⚠ HINWEIS:**  
> Bitte alle Aufgaben selbst lösen und dann über die [Lösung](solutions/solution-4/) prüfen, ob alles richtig gemacht wurde.

### Service Mesh Operator

Zuerst machen wir uns mit dem Red Hat OpenShift Service Mesh Operator (basierend auf Istio) vertraut. Dazu öffnen wir die Web Konsole in der Admin Perspektive und schauen uns den Operator unter "Installed Operators" an. Bitte nichts verändern.

Außerdem werfen wir einen Blick auf die Observability Tools **Kiali**, **Prometheus/Grafana** und **Jaeger**. Die Adressen finden wir unter Networking > Routes.

### Projekt erstellen

Wie gehabt erstellen wir zuerst ein Projekt **meshapp** mit vorangestelltem Username, also z.B. **user123-meshapp**.

Anschließend deployen wir 3 kleine Anwendungen:

```
curl -H \
  "Accept: application/vnd.github.v4.raw" \
  -L "https://api.github.com/repos/nikolaus-lemberski/openshift-modul3/contents/projects/project-4/Deployment.yml" \
  | kubectl create -f -

oc expose svc customer
```

Die Anwendungen sind drei in Reihe geschaltete apps:

1. Die "customer" app soll von außen über unsere oben erstellte _route_ aufrufbar sein
2. Die "references" app wird von "customer" gerufen und ruft den nächsten service
3. Die "recommendation" app wird von references gerufen, ist in zwei Versionen installiert und gibt neben einem einfachen Zähler noch den Hostnamen aus.

## Service Mesh


    - Projekt erstellen
    - Projekt zur ServiceMeshMemberRoll hinzufügen
    - Vorhandenes Projekt (t.b.d.: Bookinfo, istio-tutorial, eigenes Projekt?) deployen
    - Kiali öffnen und Graph analysieren
    - Jaeger öffnen und traces analysieren
    - Projekt löschen
