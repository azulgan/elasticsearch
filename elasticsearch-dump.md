# elasticsearch-dump : a tool to migrate your indexes

[elasticsearch-dump](https://github.com/taskrabbit/elasticsearch-dump) is a tool that is useful to copy all the data of one index into another one, on the same or on a different elastic search cluster.
You can also download the data into a local file or upload your data from a file, it thus enables some custom backup.

# useful tricks
If your cluster is accessible through https but with a selfsigned or a custom signed certificate, you may want to ignore certificate verification. You can then use the environment variable NODE_TLS_REJECT_UNAUTHORIZED=0 before you call elasticdump
See the [doc](https://github.com/taskrabbit/elasticsearch-dump/blob/master/README.md#bypassing-self-sign-certificate-errors)

