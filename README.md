
# Tips of GCP

## Make query strings as part of cache key for backend bucket (#CloudCDN)

According to the [doc](https://cloud.google.com/cdn/docs/caching), when using backend bucket as origin for Cloud CDN, query strings in the request URL are not included in the cache key:

> For [backend buckets](https://cloud.google.com/load-balancing/docs/backend-bucket), the cache key consists of the URI without the protocol, host, or query string.

Maybe using the query string to identify different versions of cached content is not the [best practices](https://cloud.google.com/cdn/docs/best-practices#using_versioned_urls_to_update_content) promoted by GCP. But for some legacy issues, it has to be.

So, one way to workaround this is make backend bucket to be a [static website](https://cloud.google.com/storage/docs/hosting-static-website) (do NOT enable CDN here), then use [custom origins](https://cloud.google.com/cdn/docs/custom-origins-overview) (Cloud CDN backed by [Internet network endpoint groups](https://cloud.google.com/load-balancing/docs/negs/internet-neg-concepts) backend service) which points to that static website.

For backend service, query string is part of cache key.

> For [backend services](https://cloud.google.com/load-balancing/docs/backend-service), Cloud CDN defaults to using the complete request URI as the cache key

That's it. Yes, It is tedious but works!

## How to identify the disks you mount on the GCE instance? (#ComputeEngine)

`lsblk` command [[1](https://unix.stackexchange.com/a/108951/55714)] can be used to lists information about all available or the specified block devices, output like as below wheb you issue in the GCE instance.

```
browny_lin@us-west-vm:~$ sudo lsblk -o name,mountpoint,label,size,uuid
	NAME   MOUNTPOINT LABEL           SIZE UUID
	sda                                10G
	└─sda1 /          cloudimg-rootfs  10G a6f0a496-d1bd-4df7-a548-a57b0eb41db1
```

When you attach multiple disks onto the instance, the result will show multiple devices (`sda`, `sdb`, `sdc`, ...). It's not easy to find the mapping between device names and the GCE disk names.

The better way to solve this:

When you use `gcloud compute instances attach-disk`, or you call the `attachDisk` method on the Compute API directly or via one of the SDKs, I recommend you specify the `device-name` too and set it to the disk’s name [[2](https://unix.stackexchange.com/questions/14165/list-partition-labels-from-the-command-line)].

```bash
DISK=mydisk

gcloud compute instances attach-disk ${INSTANCE} \
  --disk=${DISK} \
  --device-name=${DISK} \
  --project=${PROJECT} \
  --zone=${ZONE}
```

Then you can use `ls -l /dev/disk/by-id` to find the mapping.

## List permission differences betwenn 2 IAM roles (#IAM)

Cloud IAM is import for security. Follow principle of least privilege when binding roles to identity. But figure out the differences between 2 IAM roles is non-trivial. Below is the shortcut command to visualize the differences.

```bash
vimdiff <(gcloud iam roles describe roles/bigquery.dataEditor --format=json | jq -S '.includedPermissions') <(gcloud iam roles describe roles/bigquery.user --format=json | jq -S '.includedPermissions')
```

