# Kubernetes Questions



## No Answers
### What is KEDA? What purpose does it serve? 
### What is a node pool in AKS? 
### What is metrics server in Kubernetes? 
### What is a controller in Kubernetes? 
### What is Kustomize? 
### How do you point ConfigMaps to external file stores? 
### What is declarative syntax in Kubernetes? 

<!-- 
The answer should be completely inside of a <details></details> box, and should consist of #### and lower. Answer succinctly.  

-->



### What namespaces are passed into a template at top-level? 

<details>

More info: https://helm.sh/docs/chart_template_guide/builtin_objects/

- Release: This object describes the release itself. 
- Values: Values passed into the template from the values.yaml file and from user-supplied files. By default, Values is empty.
- Chart: The contents of the Chart.yaml file. Any data in Chart.yaml will be accessible here. For example {{ .Chart.Name }}-{{ .Chart.Version }} will print out the mychart-0.1.0.
- Subcharts: This provides access to the scope (.Values, .Charts, .Releases etc.) of subcharts to the parent. For example .Subcharts.mySubChart.myValue to access the myValue in the mySubChart chart.
- Files: This provides access to all non-special files in a chart. While you cannot use it to access templates, you can use it to access other files in the chart. See the section Accessing Files for more.
- Template: Contains information about the current template that is being executed

</details>

### What is the order of overrides for a Values.yaml? 

```
livenessProbe:
    httpGet: null
    exec: 
        command:
        - cat
        - docroot/CHANGLOG.txt
    initialDelaySeconds: 120

```

### How do you set a key in a Values.yaml to null when overriding? 






### How is default normally used for computed values?

<details>

```
drink: {{ .Values.favorite.drink | default (printf "%s-tea" (include "fullname" .)) }}
```
</details>

### Describe more about how the 'lookup' function can be used. 

### How can operators be used in templates?

For templates, the operators (eq, ne, lt, gt, and, or and so on) are all implemented as functions. In pipelines, operations can be grouped with parentheses ((, and )).

### What are the control structures, the main ones?

<details>

Helm's template language provides the following control structures:
- if/else for creating conditional blocks
- with to specify a scope
- range, which provides a "for each"-style loop

In addition to these, it provides a few actions for declaring and using named template segments:
- define declares a new named template inside of your template
- template imports a named template
- block declares a special kind of fillable template area

</details>

### Why does this cause whitespace:
```
data:
    myvalue: "Hello World"
    drink: {{ .Values.favorite.drink | repeat 5 | quote | upper }}
    food: {{ .Values.favorite.food | quote }}
    snack: {{ .Values.favorite.snack | default "chips" | quote }}
    {{ if eq .Values.favorite.drink "coffee" }}
    mug: "true"
    {{ end }}
```

### What does {{- blah  }} do ? 

### What is YAML comments vs Template Comments in Helm? What differences exist? 

### How can you access a parent scope when you have scoped into a child? 

### Explain this:
```
  toppings: |-
    {{- range .Values.pizzaToppings }}
    - {{ . | title | quote }}
    {{- end }}   
```

<details>

Sure! This snippet of Helm template code generates a list of pizza toppings, formatting each topping to have its first letter capitalized and enclosing it in quotes. Here's a step-by-step explanation:

### Explanation:

1. **`toppings: |-`**:
   - This indicates the start of a block scalar in YAML, which allows for multi-line content. The `|-` syntax means the content will be preserved exactly as written, without a trailing newline.

2. **`{{- range .Values.pizzaToppings }}`**:
   - This begins a `range` loop over the `pizzaToppings` list found in the Helm chart's `values.yaml` file.
   - `{{-` suppresses any leading whitespace/newline from the rendered output of this line.

3. **`- {{ . | title | quote }}`**:
   - `-` is the YAML syntax for starting a new list item.
   - `{{ . }}` refers to the current item in the `pizzaToppings` list.
   - `| title` converts the current item to title case (capitalizes the first letter of each word).
   - `| quote` wraps the current item in double quotes.

4. **`{{- end }}`**:
   - This ends the `range` loop.
   - `{{-` suppresses any leading whitespace/newline from the rendered output of this line.

</details>

### Why does a YAML list in a ConfigMap need to be a multi-line string?

<details>
Now, in this example we've done something tricky. The toppings: |- line is declaring a multi-line string. So our list of toppings is actually not a YAML list. It's a big string. Why would we do this? Because the data in ConfigMaps data is composed of key/value pairs, where both the key and the value are simple strings. To understand why this is the case, take a look at the Kubernetes ConfigMap docs. For us, though, this detail doesn't matter much.

</details>

### What is a tuple function in Helm?

### What are the three actions for declaring an managing templates in Helm?

<details>

In Helm, there are three primary actions for declaring and managing templates:

1. **`define`**:
   - Used to declare a new named template.
   - Syntax: `{{- define "templateName" }} ... {{- end }}`
   - Example:
     ```yaml
     {{- define "myTemplate" }}
     Hello, {{ .name }}!
     {{- end }}
     ```

2. **`include`**:
   - Used to include and render a named template within another template.
   - Syntax: `{{ include "templateName" . }}`
   - Example:
     ```yaml
     {{ include "myTemplate" . }}
     ```

3. **`template`**:
   - Similar to `include`, but with slightly different syntax and behavior. It allows you to include a named template and pass a custom context.
   - Syntax: `{{ template "templateName" . }}`
   - Example:
     ```yaml
     {{ template "myTemplate" . }}
     ```

### Detailed Explanation:

1. **`define`**:
   - This action is used to declare a reusable block of code, giving it a name that can be referenced later.
   - The template block is defined with `{{ define "templateName" }} ... {{ end }}`.

2. **`include`**:
   - This action is used to include and render a previously defined template by name.
   - The included template is rendered in the context of the current template or a specified context.
   - Useful for reusing common code snippets or components.

3. **`template`**:
   - This action also includes and renders a previously defined template.
   - It can be used interchangeably with `include`, but is sometimes preferred for its clarity in indicating the context being passed.
   - It helps in managing and passing specific contexts to the included templates.

</details>

### What are the best practices to keep in mind when naming templates?

<details>

An important detail to keep in mind when naming templates: template names are global. If you declare two templates with the same name, whichever one is loaded last will be the one used. Because templates in subcharts are compiled together with top-level templates, you should be careful to name your templates with chart-specific names.

One popular naming convention is to prefix each defined template with the name of the chart: {{ define "mychart.labels" }}. By using the specific chart name as a prefix we can avoid any conflicts that may arise due to two different charts that implement templates of the same name.

This behavior also applies to different versions of a chart. If you have mychart version 1.0.0 that defines a template one way, and a mychart version 2.0.0 that modifies the existing named template, it will use the one that was loaded last. You can work around this issue by also adding a version in the name of the chart: {{ define "mychart.v1.labels" }} and {{ define "mychart.v2.labels" }}.

</details>




### What does including the `.` do in the template import `{{- template "mychart.labels" . }}`?

<details>
Note that we pass . at the end of the template call. We could just as easily pass .Values or .Values.favorite or whatever scope we want. But what we want is the top-level scope.

</details>

### What is the difference between template and include in behavior? 

<details>

To work around this case, Helm provides an alternative to template that will import the contents of a template into the present pipeline where it can be passed along to other functions in the pipeline.

</details>

### If you have a ConfigMap, and you want to add data from an external SQL script, how can you use the .Files scope to do this? 

<details>

### How does overriding subcharts in MyChart.Values work? 

### How do global values in a Values.yaml work? What is the most common usage of this? 

### How are labels on templates globally shared? 

### How can commenting out problematic parsing sections and running helm install --dry-run --debug assist in debugging Helm charts? 

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: conf
data:
{{ (.Files.Glob "foo/*").AsConfig | indent 2 }}
---
apiVersion: v1
kind: Secret
metadata:
  name: very-secret
type: Opaque
data:
{{ (.Files.Glob "bar/*").AsSecrets | indent 2 }}
```

</details>

### What is .Files.Glob? 


### Kubernetes


### What is a StatefulSet in Kubernetes? 

<details>

A StatefulSet in Kubernetes is a type of workload resource that is used to manage stateful applications. Unlike other resources such as Deployments, StatefulSets are designed to maintain a stable identity for each of their pods and provide guarantees about the ordering and uniqueness of these pods.

### Key Features of StatefulSet:

1. **Stable, Unique Pod Identifiers**:
   - Each pod in a StatefulSet has a unique, stable network identity that persists across rescheduling.
   - Pods are named with a predictable identity, like `statefulsetname-ordinal`.

2. **Ordered, Graceful Deployment and Scaling**:
   - Pods are created in order (from `0` to `N-1`) and terminated in reverse order.
   - This ordered deployment ensures that the application can depend on the sequence and availability of the pods.

3. **Stable, Persistent Storage**:
   - StatefulSets work with PersistentVolume Claims (PVCs) to provide stable storage that persists across pod rescheduling.
   - Each pod gets a dedicated PVC, ensuring that data is not lost when pods are rescheduled.

4. **Ordered Rolling Updates**:
   - Updates to the StatefulSet are done in an ordered, controlled manner, ensuring that at any point, only one pod is being updated.
   - This behavior helps maintain the stability and availability of the application during updates.

### Use Cases:

StatefulSets are ideal for applications that require one or more of the following:
- Stable, persistent storage.
- Ordered, predictable deployment and scaling.
- Stable network identities for pods.

Typical applications include:
- Databases (e.g., MySQL, PostgreSQL)
- Distributed systems (e.g., Zookeeper, Kafka, Cassandra)
- Stateful applications that need stable network identities and persistent storage.

### Example:

Here’s an example of a StatefulSet definition for a simple web application:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "web"
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.4
        ports:
        - containerPort: 80
          name: web
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

### Explanation:
- **`serviceName`**: The name of the headless service used to manage the network identity of the pods.
- **`replicas`**: The desired number of pod replicas.
- **`selector`**: Labels used to identify the pods managed by the StatefulSet.
- **`template`**: The pod template defining the containers and other settings for the pods.
- **`volumeClaimTemplates`**: Specifies the PVCs that each pod will have, ensuring stable, persistent storage.


</details>

### How does the PVC work with StatefulSets? 

### What can the busybox image be used for in Kubernetes? 

### What does `command: ["/bin/sh", "-c"]` do?

### In my Values.yaml, if I try setting a value as the equivilant to `{{ $.Release.Name }}-my-dags-pvc`, I will get an error. However, if I wrap this in quotes, as in `"{{ $.Release.Name }}-my-dags-pvc"`, I do not. Why is this? 

### What is the Chart.lock?

### What is `helm dependency build`? 

### What are the major commands for dependencies?


### So... there's user created Helm Charts for Airflow, and ones that look like they are on the Apache website and linked, what is the difference between these two? 


<details>
`helm dependency build`
`helm dependnecy upgrade`

(Option is to use global, ie)
```
global:
  existingClaim: "{{ .Release.Name }}-my-dags-pvc"

airflow:
  dags:
    path: /opt/airflow/dags
    persistence:
      enabled: true
      existingClaim: "{{ .Release.Name }}-my-dags-pvc"
      accessMode: ReadOnlyMany

dependencies:
  - name: airflow
    version: "8.5.2"  # replace with the appropriate version
    repository: "https://airflow-helm.github.io/charts"


```

"You can use global values to ensure the subchart can access the values defined in the parent chart. This approach is often more flexible and clearer when dealing with multiple subcharts."

</details>

### Practice

'''
apiVersion: v1
kind: ConfigMap
metadata:
    name: {{ .Release.Name }}-my-dags-configmap
data:
    {{- $dags := .Files.Glob "dags/*" }}
    {{- range $name, $content := $dags }}
    {{ $name }}: |-
        {{  $content | toString | indent 8 }}
    {{- end }}
'''

The rendering of this template creates this:

'''
apiVersion: v1
kind: ConfigMap
metadata:
    name: ashflow-my-dags-configmap
data:
    dags/say_hello.py: |-
                from airflow import DAG
        from airflow.operators.python import PythonOperator
        from datetime import datetime
        
        def my_task():
            print("Hello from Kubernetes!")
        
        dag = DAG(
            'kubernetes_dag',
            schedule_interval='@daily',
            start_date=datetime(2024, 1, 1)
        )
        
        task = PythonOperator(
            task_id='say_hello',
            python_callable=my_task,
            dag=dag
        )
        
        if __name__ == "__main__":
            dag.test()
'''
Can you explain why?

### What is `ndent` vs `indent`? 

### What are `finalizers` on a PVC? What does setting this to [kubernetes.io/pvc-protection] do? How can you disable this? 



### What is a termination grace period? 

### Exercise: Create a PVC and ConfigMap. Move the data from the ConfigMap into the PVC with a Job. Run a busybox to inspect and validate that the PVC has moved the data over. 

### How do ConfigMaps work as volume mounts? Why does it use symbolic links, and what behavior does this cause? 

<details>

If the files in /dags/* are symbolic links, the cp command will copy the symbolic links themselves by default. To copy the actual files that the symbolic links point to, you can use the -L option with the cp command. The -L option tells cp to dereference symbolic links, meaning it will copy the files that the links point to rather than the links themselves.

</details>





### What if I want to preserve the directory structure after "dags? When I copy the configmap into my PVC, I want it to preserve the directory structure.  

<details>

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-dags-configmap
data:
  {{- $files := .Files.Glob "dags/**" }}
  {{- range $path, $file := $files }}
  {{ $path | replace "dags/" "" }}: |-
    {{ $.Files.Get $path | nindent 4 }}
  {{- end }}
```

</details>

### Practice q: run postgres on Kubernetses. Then create a Helm Chart.