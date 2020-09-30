.. _getting_started:

Getting started
===============
This manual is intended to provide a quick and easy way to get started with :term:`kaapana`.
This project should not be considered a finished platform or software. 
It is much more a toolkit to help you build the infrastructure you need for your specific needs.

The described steps in this tutorial will build an example :term:`platform`, which is a default configuration and contains most of the basic :term:`components<component>`.
This can be used as a starting-point to derive a customized platform, which covers your project specific requirements.

Requirements
------------
Before you get started you should be familiar with the basic concepts and components of Kaapana see :ref:`kaapana_concept`.
You should also have the following packages installed on your build-system.

1. Clone the repository:

   - :code:`git clone https://github.com/kaapana/kaapana.git`
   - :code:`git clone https://phabricator.mitk.org/source/kaapana.git`

2. Python3 

   - :code:`apt install python3 python3-pip`
   - :code:`yum install python3 python3-pip`

3. Python requirements 
   
   - :code:`python3 -m pip install -r kaapana/build-scripts/requirements.txt`

4. Docker

   - :code:`snap install docker --classic`

5. (opt) Helm

   - :code:`snap install helm --classic --channel=3.1/stable`

6. (opt) Helm-push plugin

   - :code:`helm plugin install https://github.com/chartmuseum/helm-push`

7. (opt) Helm-kubeval plugin

   - :code:`helm plugin install https://github.com/instrumenta/helm-kubeval`

.. hint::

  | **Docker as a non-root user**
  | In order to docker commands as non-root user you need to execute the following steps:
  | 1. :code:`sudo groupadd docker`
  | 2. :code:`sudo usermod -aG docker $USER`
  | 3. :code:`reboot` -> to reboot the system
  | 5. :code:`docker run hello-world` -> this should work now without root privileges
  | For more information visit the `Docker docs <https://docs.docker.com/engine/install/linux-postinstall/>`_ 

To install the platform itself, you'll also need some kind of a :term:`server` (please have a look in the Glossary for more information).


Steps needed
------------ 
To get the Kaapana-platform running, you need to execute the following steps:

1. Build and push all :term:`Dockerfiles<docker>`
2. Build and push all :term:`Helm Charts<helm>` (optional - you can use our registry)
3. Install all server requirements with the :term:`server-installation-script`
4. Deploy the platform with the :term:`platform-installation-script`

Step 1&2: Build
---------------
.. hint::

  | **Docker login needed!**
  | In order to be able to push and pull images from your registry, you need to login first.
  | For Dockerhub just use: :code:`docker login` and use your normal Dockerhub credentials.
  | For private registries you also need to specify the corresponding URL eg: :code:`docker login <URL>` 

Step 1&2 will be handeled with a build-script, which you can find it within the repository at :code:`kaapana/build-scripts/start_build.py`.

Before you start the build-process, you should have a look at the build-configuration at :code:`kaapana/build-scripts/build-configuration.yaml`.
Assuming you want to use `Dockerhub <https://hub.docker.com/>`_ as the target registry (username johndoe), then you must adjust the configuration as follows:

.. code-block:: python
   :emphasize-lines: 2,3,9,10

   http_proxy: ""
   default_container_registry: "johndoe"
   default_container_project: "" 
   default_chart_registry: "https://dktk-jip-registry.dkfz.de/chartrepo/"
   default_chart_project: "kaapana-public"
   log_level: "WARN"
   build_containers: true
   push_containers: true
   build_charts: false
   push_charts: false

As described in the :ref:`kaapana_concept`, we will utilize the DKFZ registry for Helm chart as long as there is no other easy alternative.

.. hint::

  | **Disk space needed:**
  | For the complete build of the project ~50GB of container images will be stored at :code:`/var/snap/docker/common/var-lib-docker`.


Start the build process:
:code:`python3 kaapana/build-scripts/start_build.py`

Step 3: Server Installation
---------------------------

.. hint::

  | **GPU support -> Currently only Nvidia GPUs are supported!**
  | If you want to enable GPU support, you need to install the `Nvidia drivers <https://www.nvidia.de/Download/index.aspx?lang=en>`_ first.
  | (For Ubuntu Server 20.04 :code:`sudo apt install nvidia-driver-<version>-server` should also work.)
  | Please make sure the :code:`nvidia-smi` command is working as expected!

Before the example platform "Kaapana-platform" can be deployed, all dependencies must be installed on the server first. 
To do this, you can use the :term:`server-installation-script`, which you can find at :code:`kaapana/server-installation/server_installation.sh`.
You can just copy the script to your target-system (server):

1. Make it executable: :code:`chmod +x server_installation.sh`
2. Execute the script: :code:`./server_installation.sh`
3. (opt) Enable GPU support for Microk8s :code:`./server_installation.sh -gpu`


Step 4: Platform Deployment
---------------------------

To finally deploy the platform you need to use the :term:`platform-installation-script`, which you can find at :code:`kaapana/platforms/kaapana-platform/platform_installation/install_platform.sh`.
You can just copy the script to your target-system (server) and **adjust some variables within the script**.

1. Open the :code:`install_platform.sh` script on the server
   
   :code:`nano install_platform.sh`

2. Have a look at the variables on top of the script.
   
   **You need to do at least the following customizations:**

.. code-block:: python

   ...
   CONTAINER_REGISTRY_URL="johndoe"
   CONTAINER_REGISTRY_PROJECT=""
   ...

3. Make it executable with :code:`chmod +x install_platform.sh`
4. Execute the script with :code:`./install_platform.sh`

You may be asked the following questions:

1. *Please enter the credentials for the Container-Registry:*

   Use the same credentials you used before with *docker login*

2. *Enable GPU support?*

   Answer *yes* if you have a Nvidia GPU, installed drivers and enabled GPU for Microk8s.

3. *Please enter the domain (FQDN) of the server.*

   You should enter the **domain, hostname or IP-address** where the server is accessible from client workstations.
   **Keep in mind, that valid SSL-certificates are only working with FQDN domain.**

3. *Which <platform-name> version do you want to install?:*

   Specify the version you want to install.

The script will stop and **wait** until the platform is deployed.
Since all Docker containers must be downloaded, this may take some time (~15 min).

After a successful installation you'll get the following message:

.. code-block:: python

   Installation finished.
   Please wait till all components have been downloaded and started.
   You can check the progress with:
   watch microk8s.kubectl get pods --all-namespaces
   When all pod are in the "running" or "completed" state,
   you can visit: <domain>
   You should be welcomed by the login page.
   Initial credentials:
   username: kaapana
   password: kaapana

You can now continue with the :ref:`user_guide_platform_doc`