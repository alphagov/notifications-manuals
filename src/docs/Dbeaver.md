# Dbeaver

This document describes how to connect to the local database using Dbeaver, and create a read-only connection to the produciton database. 

## Local database connection
1. Create a new connection. Use these configurations
![local-config](https://drive.google.com/uc?id=0B4iP55OVXryoTVNkOVFNNDYwWjA)
2. The tables are visible under Schemas/public/Tables
![local-config](https://drive.google.com/uc?id=0B4iP55OVXryoNklfNGptd1lKNUU)


## Create a read-only conneciton to database on server
1. You need to establish a database connection from shell. Following the procedure in this documentation, you create an SSH tunnel to our server - [PaaS Note](https://docs.google.com/document/d/1544f9LjmuZzJIdoyChUtLAgnqOUwVJGbqOHiYmcuVqo/edit#heading=h.ix67pydt9x0s)
2. From Dbeaver, make a new database connection
3. Select postgres as connection type ![Screenshot](https://docs.google.com/uc?id=0B4iP55OVXryoZlNMdmVtN1NzU28)
4. Enter the parameters and credentails for this connection. The port should be 15432 as the SSH tunnel you created in step 1. The database name and credentials can be found in Notify's credentail in /credentail/pass/postgres_logins.txt ![Screenshot](https://docs.google.com/uc?id=0B4iP55OVXryoVFhtSGFJWm1JdW8)
5. Click Next
6. Enter SSL information as this ![Screenshot](https://docs.google.com/uc?id=0B4iP55OVXryoYmdCbDhmVEVUWlE)


