# ecor/s3fs-centos

This image, based on CentOS 7, mounts zero or more S3 buckets to the file system.
Each mount point leverages caching to save bandwidth. This image was really designed
as a base image on which other images can be built. It must be run as a privileged
container, and the mounts will be accessible to the entire system. It expects the
following environment variables:

- **AWS_KEY**: Your AWS key.
- **AWS_SECRET**: Your AWS secret.
- **AWS_BUCKET**: This can be a space delimited list of bucket names.
- **AWS_MOUNT_PREFIX** _(Optional)_: This is the base directory where the buckets will be mounted.
By default, this is `/S3`. If `AWS_BUCKET` is set to `my.bucket.a my.bucket.b`, then
two mounts will be created and accessible at `/S3/my.bucket.a` and `/S3/my.bucket.b`.

The key/secret must have a minimum of `read` and `list` permissions on the buckets in
order to work. `write` permissions are also required if you want to copy anything to the
mounted directories.

## Testing the Container

To test the system, run an ephemeral container:

```sh
docker run --rm --name s3fs \
-e AWS_KEY=MYKEY \
-e AWS_SECRET=MYSECRET \
-e AWS_BUCKET="my.first.bucket my.second.bucket" \
-e AWS_MOUNT_PREFIX=/S3 \
--privileged \
ecor/centos-s3fs
```

This should render output like:

```sh
Mounted AWS Bucket: /S3/my.first.bucket
Mounted AWS Bucket: /S3/my.second.bucket
```

## Building on the Container

Building on this container is like any other, but this won't do anything until the mount
is initated. A command called `mountS3` (available at `/usr/bin/mountS3`) can be called in
the child container when the file system should be mounted. This needs to be done in the
`CMD` or `ENTRYPOINT` command, because this is the only time Docker has the permissions
required to mount a drive.

**Examples:**

```sh
CMD ["mountS3"]
```

```sh
ENTRYPOINT ["/usr/bin/mountS3"]
```

## Contributions

I'd love to have an alpine linux-based version of this instead of CentOS if someone has time.
