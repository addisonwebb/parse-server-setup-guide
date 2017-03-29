# Setup MongoDB

### Create New database
1. In the MongoDB Deployments section click "Create new"
  - Cloud Provider: AWS
  - Plan: Single-Node, Sandbox (free)
  - Database Name: `test-goexploremichigan`
2. Setup a DB user under the Users tab
  - Make sure you save the username and password in a password manager!
  - db passwords work best when they don't include special characters
3. You will use the DB URI for the `PARSE_SERVER_DATABASE_URI` environment variable
  - Example `mongodb://<dbuser>:<dbpassword>@ds123456.mlab.com:45380/test-goexploremichigan`
4. Logout

### [<- Previous: Parse Server Intro](./readme.md) --- [Next: Setup Parse Server ->](./setup-parse-server.md)
