# Exercise: Hello World

1. Create a directory for your bundle.

    ```
    mkdir Hello
    ```

2. Scaffold a bundle:

    ```
    porter create
    ```

3. Open porter.yaml and edit the Docker Registry for your bundle:

    ```
    invocationImage: <YOUR_DOCKERHUB>/porter-azure-wordpress:latest
    ```

4. Build your bundle:

    ```
    porter build
    ```

5. Install your bundle!

    ```
    duffle install HELLO1 -f bundle.json --insecure
    ```
