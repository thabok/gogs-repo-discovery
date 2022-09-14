# gogs-repo-discovery
A seed job example that shows how to discover gogs repositories and auto-create jenkins pipeline jobs for them, including webhooks for push-event triggers

This example assumes:
 - a secret-text credential called 'gogs-token-thabok' with the value 'token ********************************'
 - the gogs server to be found at 192.168.19.112:3000
 - you're looking for repositories of the user 'thabok' inside gogs
 - the gogs-webhook plugin is installed for Jenkins for the webhooks to work
