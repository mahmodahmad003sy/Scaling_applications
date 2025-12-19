2. Docker Registry for Linux — Part 1
   i used This command which proves the image was stored in the external (host-mounted) registry folder, not just inside the container.
   curl http://127.0.0.1:5000/v2/_catalog
  the result: 
  <img width="492" height="592" alt="image" src="https://github.com/user-attachments/assets/b93879e6-6f2c-48a4-9988-b838bce09b01" />
================================================================================================================================================
3-  
  A) Proof it’s HTTPS
    curl -I http://<127.0.0.1:5000>/v2/   =>fail
    curl -k -I http://<127.0.0.1:5000>/v2/   =>401 Unauthorized
    
  <img width="431" height="305" alt="image" src="https://github.com/user-attachments/assets/7979c339-b863-49ff-a8b4-cd9365f82851" />

  B)  curl -k -u moby:WRONGPASS https://<registry-host>/v2/_catalog    =>401 Unauthorized
      curl -k -u moby:gordon https://<registry-host>/v2/_catalog    =>Login Succeeded
      <img width="933" height="177" alt="image" src="https://github.com/user-attachments/assets/ef02796c-b4aa-457e-9006-223babd9c3cb" />
      ================================================================================================================================================
4-
  A)Nodes when everything is Active
    docker node ls
    <img width="893" height="172" alt="image" src="https://github.com/user-attachments/assets/f75a4464-d526-49a6-97ef-63a1a2b098d6" />
    
  B) Put one node into Drain, then show nodes again
     docker node update --availability drain node2
     docker node ls
     <img width="886" height="148" alt="image" src="https://github.com/user-attachments/assets/b1d77f50-d3b7-4272-ab30-469bccd3eeee" />

C) Prove tasks moved away from the drained node
    docker service ps <SERVICE_NAME>
    docker node ps <DRAINED_NODE_NAME>
    <img width="1018" height="218" alt="image" src="https://github.com/user-attachments/assets/38926939-8048-48d5-8f07-b86cc44429bb" />
D) Restore the node back to Active
   docker node update --availability active <NODE_NAME>
   docker node ls
   <img width="917" height="187" alt="image" src="https://github.com/user-attachments/assets/0d2d7c9f-afcb-4d0c-b1b5-52d08d670d2d" />
E) Show whether the service came back to that node or not
  docker service ps <SERVICE_NAME>.
  <img width="1060" height="235" alt="image" src="https://github.com/user-attachments/assets/9cca8b84-56fe-4026-b338-7465ccfa3758" />
  
 4-1: Did the service resume on that node after Drain → Active?
   No.
When you set the node to Drain, Swarm reschedules its tasks to other nodes. When you switch it back to Active, Swarm does not automatically move tasks back to that node unless there is a reason (e.g., scaling, restart, rebalance).

4-2:What must be done to run the service on that node again?
   answer 1:You need to trigger scheduling onto that node, for example:

            Scale the service up, then back down (so new tasks may land there):
            
                docker service scale <SERVICE_NAME>=<BIGGER_NUMBER> then reduce
            
            Force an update so tasks restart and may be placed on that node:
            
                docker service update --force <SERVICE_NAME>
            
            Or set a placement constraint to require running on that node (advanced):
            
              docker service update --constraint-add node.hostname==<NODE_NAME> <SERVICE_NAME>
              
    or we can: docker service update --force
      

================================================================================================================================================
5 ) shows 2 nodes, one Leader, one Active.
   <img width="965" height="228" alt="image" src="https://github.com/user-attachments/assets/5065e79f-fe44-4268-8089-bd32a2d59679" />


   config (replicas + healthcheck):
   <img width="930" height="410" alt="image" src="https://github.com/user-attachments/assets/699f6129-96e1-47e2-b823-1a24bf7f30f0" />

   output of deploy command:
   docker stack deploy -c docker-stack.yml mystack
   <img width="922" height="216" alt="image" src="https://github.com/user-attachments/assets/f52ac77a-d01f-4261-80ed-448753b8192d" />

   Prove replicas are applied:
   docker stack services mystack
   <img width="945" height="169" alt="image" src="https://github.com/user-attachments/assets/c17eb6f7-fa6f-458f-87d7-330ad0d217ca" />

   Show tasks and where they run+Prove healthcheck is configured
   <img width="976" height="395" alt="image" src="https://github.com/user-attachments/assets/06d2e08d-602d-4f4d-b2bc-f14549cba99d" />



  
