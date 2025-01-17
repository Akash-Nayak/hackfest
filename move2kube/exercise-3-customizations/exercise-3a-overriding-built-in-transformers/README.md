# Customizing generated Dockerfile and behavior of built-in transformer

## Steps for Move2Kube CLI/UI

1. Run Move2Kube transform on the `enterprise-app` without any customizations and observe the output:

    ```console
    $ cd exercise-3-customizations/exercise-3a-overriding-built-in-transformers
    ```

    `--qa-skip` option allows us to run Move2Kube transform non-interactively to skip the questions and use the default answers.

    ```console
    $ move2kube transform -s ../../source/enterprise-app/ --qa-skip --overwrite
    ```

    If you are using the Move2Kube UI, create a new workspace and create a new project inside the workspace. Now, keep the the Project input type as Source folder and upload the [enterprise-app.zip](https://github.com/Akash-Nayak/hackfest/blob/main/move2kube/source/enterprise-app.zip) file. After the planning completes, click on the Start Transformation button.

2. List the files in the frontend service directory of the `enterprise-app`:

    ```console
    $ ls myproject/source/frontend
    ```

    As you notice, there are no scripts named `start-nodejs.sh` in the frontend service directory.
    
    ```console
    Dockerfile      __mocks__       jest.config.js    package-lock.json    src               test-setup.js         webpack.dev.js
    LICENSE         dist            manifest.yml      package.json         stories           tsconfig.json         webpack.prod.js
    README.md       dr-surge.js     nodemon.json      server.js            stylePaths.js     webpack.common.js
    ```

3. Check the generated Dockerfile for frontend service of the `enterprise-app`:

    ```console
    $ cat myproject/source/frontend/Dockerfile
    ```

    If we notice the Dockerfile generated for the frontend app, it uses registry.access.redhat.com/ubi8/nodejs-14 as base image.
    
    ```console
    FROM registry.access.redhat.com/ubi8/nodejs-14
    COPY . .
    RUN npm install
    RUN npm run build
    USER root
    RUN chown -R 1001:0 /opt/app-root/src/.npm
    RUN chmod -R 775 /opt/app-root/src/.npm
    USER 1001
    EXPOSE 8080
    CMD npm run start
    ```

4. List the deployment artifacts generated by Move2Kube:

    ```console
    ls myproject/deploy
    ```

    If you notice, the Kubernetes yamls are generated in myproject/deploy/yamls directory.
    
    ```console
    cicd                  compose               knative               knative-parameterized yamls                 yamls-parameterized
    ```

5. Lets customize the generated output. We will use a [custom configured version](https://github.com/konveyor/move2kube-transformers/tree/main/custom-dockerfile-change-built-in-behavior) of the [nodejs built-in transformer](https://github.com/konveyor/move2kube/tree/main/assets/built-in/transformers/dockerfilegenerator/nodejs) and [kubernetes built-in transformer](https://github.com/konveyor/move2kube/tree/main/assets/built-in/transformers/kubernetes/kubernetes) to achieve this. We already have these in this exercise in the `customizations` folder.

6. Run Move2Kube transform with the customizations, specifying the customization using the -c flag.

    ```console
    $ move2kube transform -s ../../source/enterprise-app/ -c customizations/ --qa-skip --overwrite
    ```

    If you are using the Move2Kube UI, create a new workspace and create a new project inside the workspace. Now, keep the the Project input type as Source folder and upload the [enterprise-app.zip](https://github.com/Akash-Nayak/hackfest/blob/main/move2kube/source/enterprise-app.zip) file. Now, keep the Project input type as Customization folder and upload the [customizations.zip](./customizations.zip). Click on the Start Planning button. After the planning completes, click on the Start Transformation button.

7. Check the modified Dockerfile for frontend service of the `enterprise-app`:

    ```console
    $ cat myproject/source/frontend/Dockerfile
    ```

    Observe the base image is changed from `registry.access.redhat.com/ubi8/nodejs-14` to `quay.io/konveyor/nodejs-12` in the generated nodejs dockerfile. The CMD instruction in the generated dockerfile is modified to use the `start-nodejs.sh` file.

    ```console
    FROM quay.io/konveyor/nodejs-12
    COPY . .
    RUN npm install
    RUN npm run build
    EXPOSE 8080
    CMD sh start-nodejs.sh
    ```

6. List the files in the frontend service directory of the `enterprise-app`:

    ```console
    $ ls myproject/source/frontend
    ```

    A new `start-nodejs.sh` file is added to `frontend` service directory which will be used in the CMD dockerfile instruction.

    ```console
    Dockerfile        __mocks__         jest.config.js    package-lock.json src               stylePaths.js     webpack.common.js
    LICENSE           dist              manifest.yml      package.json      start-nodejs.sh   test-setup.js     webpack.dev.js
    README.md         dr-surge.js       nodemon.json      server.js         stories           tsconfig.json     webpack.prod.js
    ```


8. List the kubernetes deployment artifacts generated by Move2Kube:

    ```console
    ls myproject/
    ```

    The default location of the kubernetes yamls that are generated changed from `myproject/deploy/yamls` to `myproject/yamls-elsewhere`.

    ```console
    Readme.md                     scripts                       yamls-elsewhere
    deploy                        source                        yamls-elsewhere-parameterized
    ```

For more detailed instructions, you can check the full tutorial here - https://move2kube.konveyor.io/tutorials/customizing-the-output/custom-dockerfile-change-built-in-behavior
