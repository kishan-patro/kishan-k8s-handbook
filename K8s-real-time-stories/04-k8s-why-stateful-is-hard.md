# Why Stateful is Hard in Kubernetes

Kubernetes is beautiful until you run stateful workloads.
Your lead just asked you to run Postgres on Kubernetes.

You deploy Postgres as a Deployment. Three replicas, apply it, Postgres starts. You connect, insert some rows, everything works.

Then the pod crashes and restarts. You connect and your data is gone.

The container filesystem is ephemeral. It exists only for the lifetime of the container. When the container dies, the filesystem dies with it.

Losing your database on every restart is not an edge case. It is the default behavior.

So you introduce a PersistentVolume and a PersistentVolumeClaim. Storage that exists independently of the pod.

The pod crashes, a new one starts, reattaches to the same volume, and your data is still there.

Developers declare what they need and Kubernetes provisions it automatically. No ticket, no wait, no cloud console.

Ephemeral storage lost your data. PersistentVolume and PVC fixed that.

But your database is still broken.

You attach the PVC to your Deployment with three Postgres replicas expecting high availability.

Instead you get corruption. All three pods write to the same disk simultaneously with no coordination. They conflict, they corrupt, the database breaks.

The problem is not the storage. The problem is the Deployment.

A Deployment treats every pod as identical. Start them in any order, kill them in any order, every pod is the same. Postgres does not work like that. Pod 0 is the primary. It accepts writes.

Pod 1 and Pod 2 serve reads and follow the primary. If Pod 0 crashes and comes back with a random name, Postgres has no idea who the primary is anymore.

So you replace the Deployment with a StatefulSet. Pod 0 is always Pod 0. It starts first, shuts down last, and comes back with the same name, the same role, the same identity.

Each pod gets its own dedicated volume. No sharing, no conflicts, no corruption.

Deployments could not handle stateful workloads. StatefulSet fixed that.

Your database is stable. Now you have three environments.

Dev, staging, and production each need different configs. So you copy the YAML, change the values, apply. Repeat for staging. Repeat for production. Over time those files diverge.

Someone updates production and forgets staging. Small differences accumulate until debugging becomes guesswork.

So you introduce Helm. Write your YAML once as a template. Each environment gets its own values file. One source of truth.

Zero drift. When something breaks after a Postgres upgrade, one command rolls back.

Duplicated YAML drifted apart. Helm fixed that.

Each concept exists because the previous one left something broken.
