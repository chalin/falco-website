---
title: Kernel Events
description: 'Events related to the Kernel tells us most of what happens above.'
linktitle: Kernel Events
weight: 20
aliases:
  - /docs/event-sources/drivers
---

Falco uses different instrumentations to analyze the system workload and pass security events to userspace. We usually refer to these instrumentations as **syscall sources** since the generated events are strictly related to the syscall context.

There are different supported syscall sources:

- Kernel module *(default)*
- eBPF probe
- modern eBPF probe

|             | Kernel module | eBPF probe | Userspace       | modern eBPF probe |
| ----------- | ------------- | ---------- | --------------- | ----------------- |
| **x86_64**  | >= 2.6        | >= 4.14    | No requirements | [Minimal set of features](/docs/event-sources/drivers/#requirements) |
| **aarch64** | >= 3.4        | >= 4.17    | No requirements | [Minimal set of features](/docs/event-sources/drivers/#requirements) |

## Kernel module

By default, the kernel module will be installed when installing the Falco [debian/rpm](/docs/getting-started/installation) package, when running the `falco-driver-loader` script shipped within the [binary package](/docs/getting-started/installation#linux-binary), or when running the `falcosecurity/falco-driver-loader` docker image (that just wraps the aforementioned script).

To install the kernel module, please refer to the [installation](/docs/getting-started/installation/#install-driver) page.

## eBPF probe

The eBPF probe is an alternative source to the one described above.

To install the eBPF probe, please refer to the [installation](/docs/getting-started/installation/#install-driver) page.

To enable the eBPF support in Falco set the `FALCO_BPF_PROBE` environment variable to an empty value (ie. `FALCO_BPF_PROBE=""`, in this way Falco will search under `$HOME/.falco` directory for a file called `falco-bpf.o`) or otherwise explicitly set it to the path where the eBPF probe resides.

## Modern eBPF probe

The modern probe is an alternative driver for Falco. The main advantage it brings to the table is that it is embedded into Falco, which means that you don't have to download or build anything, if your kernel is recent enough Falco will automatically inject it!

### What's new

The new probe is highly customizable, you are not obliged to use one buffer [for each CPU](https://github.com/falcosecurity/falco/blob/660da98e4c37f4d4f79ec4bebf4379d9b90b0892/falco.yaml#L292) you can also use just one huge buffer for all your CPUs! And obviously, also the [buffer size](https://github.com/falcosecurity/falco/blob/660da98e4c37f4d4f79ec4bebf4379d9b90b0892/falco.yaml#L226) is customizable! All this is possible thanks to new outstanding features like [the CO-RE paradigm](https://nakryiko.com/posts/bpf-portability-and-co-re/), [the BPF ring buffer](https://nakryiko.com/posts/bpf-ringbuf/) and many others, if you are curious you can read more about them in this [blog post](/blog/falco-modern-bpf#what-s-new).

### Requirements

The modern BPF probe doesn't require a specific kernel version. Usually, all versions `>=5.8` are enough but there are cases in which the required features could also be backported into older kernels, so it wouldn't be completely fair to define `5.8` as the first supported version. The 2 main required features are:

1. [BPF ring buffer](https://www.kernel.org/doc/html/next/bpf/ringbuf.html) support.
2. A kernel that exposes [BTF](https://docs.kernel.org/bpf/btf.html).

Falco can detect if these features are available on the running machine and can notify you if something is missing. You only have to run Falco and see what happens!

### How to run it

The modern eBPF probe supports all the installation methods of other drivers:

* [Falco packages](/docs/getting-started/installation/#installation-with-dialog)
* [Falco binary](/docs/getting-started/running/#falco-binary)
* [Docker](/docs/getting-started/running/#modern-ebpf)
* [Helm chart](https://github.com/falcosecurity/charts/blob/master/falco/README.md#daemonset)

### Useful resources

* [Modern BPF blog post](/blog/falco-modern-bpf/)
* [Modern BPF proposal](https://github.com/falcosecurity/libs/blob/master/proposals/20220329-modern-bpf-probe.md)
* [eBPF day presentation](https://youtu.be/BxoKztfHnYY)

## Userspace instrumentation (no more maintained)

{{% pageinfo color="warning" %}}
[The pdig project](https://github.com/falcosecurity/pdig) is currently in archived status. Falco still has support for userspace instrumentation, but `pdig` is not actively maintained currently.
{{% /pageinfo %}}

Differently from the other drivers, as the name suggests, userspace instrumentation happens 100% in userspace.

The Falco community, in 0.24.0 promoted the userspace instrumentation feature to **[official support](https://github.com/falcosecurity/evolution#official-support)** to be included in Falco.

However, there's a difference between userspace instrumentation and the other drivers. At the moment of writing, the Falco project does not have any officially supported userspace instrumentation implementation.

To summarize: the code that defines the contracts to do userspace instrumentation in Falco itself is stable and under **[official support](https://github.com/falcosecurity/evolution#official-support)**. While there's
no implementation that reached the status of **[official support](https://github.com/falcosecurity/evolution#official-support)** yet

The community is working on an implementation based on `PTRACE(2)` that you can find [falcosecurity/pdig](https://github.com/falcosecurity/pdig).

How to enable userspace instrumentation in Falco:

- Start Falco with the `--userspace` flag. This will tell Falco to look at userspace instrumentation instead of looking at the Kernel module (the default).
- Select the userspace instrumentation implementation you want to use (let's take [pdig](https://github.com/falcosecurity/pdig) as it is the only one available now)
- Now, since `pdig` is still [incubating](https://github.com/falcosecurity/evolution#incubating) - a very early stage - it's not included in our [release process](https://github.com/falcosecurity/falco/blob/master/RELEASE.md). This, for you, means that to install it, you will need to compile it yourself. Follow the instructions [here](https://github.com/falcosecurity/pdig#instructions).
- Now that you have `pdig` installed, you will need to start it. Remember, in the case of `pdig`, it does not know the root process tree you want to instrument by itself. You have to specify that via the  `-p` flag or start the process with `pdig` itself. It's very similar to running a debugger against a binary. You can run a process directly using it or instrument an existing process. Full instructions [here](https://github.com/falcosecurity/pdig#how-to-run-it).

There are attempts at making the installation process easier, but the community didn't pick one yet and it's likely that many iterations will go into making any one of the below GA. However, here's a list of projects you can look at if you want to get Falco with userspace instrumentation in Kubernetes.

- [Falco Trace](https://github.com/kris-nova/falco-trace) - a convenient container image to use as a base to add userspace instrumentation to your images.
- [Falco Inject](https://github.com/fntlnz/falco-inject) - a tool that uses Falco Trace artifacts to inject Falco and userspace instrumentation into your containers via the Kubernetes API.

As you probably already understood, the userspace instrumentation drivers are a bit different, handle them with care!
