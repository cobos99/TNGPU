# DIPC Cluster Summary

**Author:** Jesús Cobos Jiménez

See [DIPC official documentation](https://scc.dipc.org/docs/) for complete information.

## Logging in into the cluster

There are [two clusters](https://scc.dipc.org/docs/general/overview/) available. We are mostly interested on the `Atlas EDR` cluster, which is the one with GPUs available. To log into this cluster, one uses `ssh` from terminal.

```bash
ssh username@atlas-fdr.sw.ehu.es
```

**Note** that on first log in you will be prompted to change your default password ([See here](https://scc.dipc.org/docs/access/accounts/#modifying-your-password)).

Here, we have access to a Linux shell, which should not be used to directly execute our jobs, but to manage our files and submit our jobs using the job manager `SLURM`.

## Uploading/Downloading files to the cluster

We have access to two main roots of the file system of the cluster, namely `/dipc/username/` and `/scratch/username/`. The first is path `/dipc/` is where we are intended to save large files that we want to save for long as it is backed up regularly. The second path points to high performance storage, and it is from where we should submit our jobs. My recommendation is to use temporarily `/scratch/` when submitting jobs and then downloading the results directly to your computer.

The best way to manage the files on the cluster is to use [FileZilla](https://filezilla-project.org/), which is a very convenient FTP file explorer. Configuring FileZilla to access the cluster is very simple

![Image](./Summary figures/Server_add_button.pngraw=true)

