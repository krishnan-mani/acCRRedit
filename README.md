ac-[CRR](http://docs.aws.amazon.com/AmazonS3/latest/dev/crr.html)-edit is a play on the acronym "CRR" which stands for "Cross Region Replication" with Amazon S3. This was first created to help test S3 CRR for a mix of files.

See [HOW-TO](HOW-TO.md) for instructions.

TODO
====

(Feel free to create an issue)

- [ ] Add support for SSE on bucket using KMS
- [ ] Provision an S3 VPC endpoint so the instance fleet can just run within a private subnet with minimal network access
- [ ] Must verify that the inventory is configured correctly and works!
- [ ] support a mix of files by file size, file count, and file organisation by folders and depth of nesting
- [ ] calculate fleet size, etc. based on total size of files to be generated
- [ ] use multi-part upload and other speedier ways to upload large files to s3
- [ ] provide a way to estimate costs, including an upper bound on costs
- [ ] add links in documentation to pricing
- [ ] tear down spot instance fleets after writing file payloads
