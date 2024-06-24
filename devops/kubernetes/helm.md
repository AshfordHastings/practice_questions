# Helm Questions

## Some

### How does pipelining work when a function takes two arguments? 

<details>

In Helm templating, when pipelining is used with a function that takes two arguments, the first argument is typically the output of the previous function or a direct value, and the second argument is provided directly in the function call.

When pipelining arguments like this, the result of the first evaluation (.Values.favorite.drink) is sent as the last argument to the function.

#### Example:

Consider the `cat` function which concatenates two strings:

```yaml
{{ "World" | cat "Hello " }}
```

Here, `"World"` is piped into the `cat` function as the first argument, and `"Hello "` is provided as the second argument. The result is:

```yaml
Hello World
```

**Explanation**:
1. `"World"` is the output of the initial value.
2. `cat` function takes this output as its first argument.
3. `"Hello "` is the second argument provided directly to `cat`.

#### Another Example:

Using `default` with a pipeline:

```yaml
{{ .Values.someValue | default "defaultValue" }}
```

Here, `.Values.someValue` is the output piped into the `default` function as the second argument, and `"defaultValue"` is the first argument.

</details>

### What are some of the most common functions in Helm?

<details>

Some of the most common functions in Helm are:

1. **`quote`**: Quotes a string.
   - `{{ quote "Hello World" }}`
   - Output: `"Hello World"`

2. **`upper`**: Converts a string to uppercase.
   - `{{ upper "hello" }}`
   - Output: `HELLO`

3. **`lower`**: Converts a string to lowercase.
   - `{{ lower "HELLO" }}`
   - Output: `hello`

4. **`default`**: Provides a default value if the given value is empty.
   - `{{ default "defaultValue" .Values.someValue }}`
   - Output: `defaultValue` (if `.Values.someValue` is empty)

5. **`tpl`**: Renders a template inside another template.
   - `{{ tpl .Values.templateString . }}`
   - Used to render a template string stored in values.

6. **`required`**: Fails template rendering if the provided value is missing or empty.
   - `{{ required "Value is required" .Values.someValue }}`
   - Throws an error if `.Values.someValue` is empty.

7. **`include`**: Includes another template.
   - `{{ include "mytemplate" . }}`
   - Includes and renders the named template.

8. **`toYaml`**: Converts a value to YAML format.
   - `{{ toYaml .Values.someMap }}`
   - Useful for formatting values as YAML.

9. **`lookup`**: Looks up resources in the Kubernetes cluster.
   - `{{ lookup "v1" "Service" "default" "myservice" }}`
   - Returns the specified resource if it exists.

10. **`hasKey`**: Checks if a map contains a given key.
    - `{{ hasKey .Values "someKey" }}`
    - Returns `true` if the key exists.

11. **`b64enc`**: Encodes a string in base64.
    - `{{ b64enc "hello" }}`
    - Output: `aGVsbG8=`

12. **`b64dec`**: Decodes a base64-encoded string.
    - `{{ b64dec "aGVsbG8=" }}`
    - Output: `hello`

</details>

### What is the difference in syntax between functions and pipelines in Helm templating? 

<details>
In Helm templating, functions and pipelines have distinct syntaxes and usages:

1. **Functions**:
   - Functions in Helm are similar to those in programming languages.
   - They are called by using the `{{ <function_name> <arguments> }}` syntax.
   - For example: `{{ quote "Hello World" }}`

2. **Pipelines**:
   - Pipelines are used to chain multiple functions together, passing the output of one function as the input to the next.
   - The syntax uses the pipe `|` character to connect functions.
   - For example: `{{ "Hello World" | quote | upper }}`

</details>

### What are the referenceable parameters on the .Release object? 

<details>

- Release: This object describes the release itself. It has several objects inside of it:
- Release.Name: The release name
- Release.Namespace: The namespace to be released into (if the manifest doesn’t override)
- Release.IsUpgrade: This is set to true if the current operation is an upgrade or rollback.
- Release.IsInstall: This is set to true if the current operation is an install.
- Release.Revision: The revision number for this release. On install, this is 1, and it is incremented with each upgrade and rollback.
- Release.Service: The service that is rendering the present template. On Helm, this is always Helm.
</details>


###

###

###

###

###


### What are the standard practices for where to put templates?

<details>
- Most files in templates/ are treated as if they contain Kubernetes manifests
- The NOTES.txt is one exception
- But files whose name begins with an underscore (_) are assumed to not have a manifest inside. These files are not rendered to Kubernetes object definitions, but are available everywhere within other chart templates for use.

These files are used to store partials and helpers. In fact, when we first created mychart, we saw a file called _helpers.tpl. That file is the default location for template partials.

</details>

### How can you use the `base` function from Go's path package when accessing the Files object in Helm? What other functions are avaliable? 

### How does namespacing work in Helm? How do you handle multinamespaced deployments? 

### How can Helm Hooks be used to allow for dependent jobs to be ran? 

helm template --dry-run --debug wiki-dags .