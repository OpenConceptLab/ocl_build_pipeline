## Build GoCD Environment
GoCD build pipeline configuration for oclapi and ocl_web

### How to build GoCD Server?
You can follow the steps below for building GoCD Server.

   * ``` docker pull gocd/gocd-server:17.2.0 ```
   * ``` docker run -d -p 8153:8153 -p 8154:8154 gocd-server ```

### How to build GoCD Agent?
   * ``` git clone git@github.com:OpenConceptLab/go-cd.git ```
   * ``` docker build -t go-agent .```
   * ``` docker run -d --name go-agent --hostname go-agent -e GO_SERVER_URL=https://$IP:8154/go go-agent ```

After these steps;
We recommend you to use least two go-agent.
   
   * You should copy ssh-key from go-cd agent to go-cd server. For that;
   	    * You should enter to go-agent container.
	        * ``` docker exec -it go-agent bash  ```
	      * You should do after that operation with called go user.
            * ``` su go  ```
        * You should generate ssh-key.
            * ``` ssh-keygen -t rsa  ```
        * You can send ssh-key to go-cd server.
            * ``` ssh-copy-id -i $USER@$IP  ```





