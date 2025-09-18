Related Issue: [#11228](https://github.com/longhorn/longhorn/issues/11228)
---

## Managing Longhorn with CLI

The following sections will guide you through managing Longhorn's space usage using `kubectl` and by manipulating Custom Resources (CRs). This method is ideal for automation and scripting, providing an alternative to the Longhorn UI.

### 1. Viewing Cluster-Wide Space Usage

The Longhorn UI dashboard aggregates space usage from all nodes. You can replicate this view by querying the `nodes.longhorn.io` CRs and parsing their status fields.

Run the following command to get a summary of disk usage on all your nodes:

```bash
kubectl get nodes.longhorn.io -n longhorn-system
```

**Output**:

```bash
NAME          READY   ALLOWSCHEDULING   SCHEDULABLE   AGE
ubuntu-lh-2   True    true              True          6d3h
```

From this output, you will see a list of your nodes and their status. To get more detailed space information, we will use a custom-columns query.

Run this command to get a more detailed, table-formatted view of disk metrics:

```bash
kubectl get nodes.longhorn.io -n longhorn-system -o custom-columns=NODE:.metadata.name,TOTAL_SIZE:.status.diskStatus.*.storageMaximum,AVAILABLE_SIZE:.status.diskStatus.*.storageAvailable
```

**Output**:

```bash
NODE          TOTAL_SIZE    AVAILABLE_SIZE
ubuntu-lh-2   51409092608   39950745600
```

### 2. Viewing Space Usage of Each Node

The "Nodes" page in the UI provides granular details about space on each disk of a specific node. You can access this same data by running a `kubectl get` command on an individual node's CR.

Replace `<node-name>` with the name of one of your nodes (for example, `ubuntu-lh-2` in this case) and run this command:

```bash
kubectl get nodes.longhorn.io <node-name> -n longhorn-system -o yaml
```

**Output**:

```yaml
apiVersion: longhorn.io/v1beta2
kind: Node
metadata:
  creationTimestamp: "2025-09-09T08:21:01Z"
  finalizers:
  - longhorn.io
  generation: 1
  name: ubuntu-lh-2
  namespace: longhorn-system
  resourceVersion: "5343"
  uid: 3e51ac89-6374-4432-971c-69578d70ef6a
spec:
  allowScheduling: true
  disks:
    default-disk-4c31e9a428aa4512:
      allowScheduling: true
      diskDriver: ""
      diskType: filesystem
      evictionRequested: false
      path: /var/lib/longhorn/
      storageReserved: 15422727782
      tags: []
  evictionRequested: false
  instanceManagerCPURequest: 0
  name: ubuntu-lh-2
  tags: []
status:
  autoEvicting: false
  conditions:
  - lastProbeTime: ""
    lastTransitionTime: "2025-09-09T08:21:01Z"
    message: Node ubuntu-lh-2 is ready
    reason: ""
    status: "True"
    type: Ready
  - lastProbeTime: ""
    lastTransitionTime: "2025-09-09T08:21:01Z"
    message: ""
    reason: ""
    status: "True"
    type: Schedulable
  - lastProbeTime: ""
    lastTransitionTime: "2025-09-09T08:21:01Z"
    message: 'Missing packages: [cryptsetup]'
    reason: PackagesNotInstalled
    status: "False"
    type: RequiredPackages
  - lastProbeTime: ""
    lastTransitionTime: "2025-09-09T08:21:01Z"
    message: ""
    reason: ""
    status: "True"
    type: Multipathd
  - lastProbeTime: ""
    lastTransitionTime: "2025-09-09T08:21:01Z"
    message: ""
    reason: ""
    status: "True"
    type: NFSClientInstalled
  - lastProbeTime: ""
    lastTransitionTime: "2025-09-09T08:21:01Z"
    message: Kernel modules [dm_crypt] are not loaded
    reason: KernelModulesNotLoaded
    status: "False"
    type: KernelModulesLoaded
  - lastProbeTime: ""
    lastTransitionTime: "2025-09-09T08:21:31Z"
    message: ""
    reason: ""
    status: "True"
    type: MountPropagation
  diskStatus:
    default-disk-4c31e9a428aa4512:
      conditions:
      - lastProbeTime: ""
        lastTransitionTime: "2025-09-09T08:21:31Z"
        message: Disk default-disk-4c31e9a428aa4512(/var/lib/longhorn/) on node ubuntu-lh-2
          is ready
        reason: ""
        status: "True"
        type: Ready
      - lastProbeTime: ""
        lastTransitionTime: "2025-09-09T08:21:31Z"
        message: Disk default-disk-4c31e9a428aa4512(/var/lib/longhorn/) on node ubuntu-lh-2
          is schedulable
        reason: ""
        status: "True"
        type: Schedulable
      diskDriver: ""
      diskName: default-disk-4c31e9a428aa4512
      diskPath: /var/lib/longhorn/
      diskType: filesystem
      diskUUID: 8c4327c0-9a19-411f-ade1-89fcc85faed6
      filesystemType: ext2/ext3
      instanceManagerName: instance-manager-d3445a46aab28601b5cb4477c5f6d1a2
      scheduledBackingImage: {}
      scheduledReplica: {}
      storageAvailable: 39950745600
      storageMaximum: 51409092608
      storageScheduled: 0
  region: ""
  snapshotCheckStatus: {}
  zone: ""
```

In the output, you will see a `status` section with detailed metrics for each disk on that node.
- The **Size** (max actual available space) is listed under `status.diskStatus.<disk-name>.storageMaximum`.
- The **Allocated** space (total volume size) is under `status.diskStatus.<disk-name>.storageScheduled`.
- The **Used** space (actual data stored on disk) is calculated by subtracting `storageAvailable` from `storageMaximum`.

### 3. Updating Storage Over Provisioning Percentage

This is a key global setting that can be changed to control how much over-provisioning is allowed. The Longhorn UI provides a form to update this value, but you can do it via the CLI using a `kubectl patch` command.

**Command** (View Current Setting):

First, let's view the current value of the `storage-over-provisioning-percentage` setting:

```bash
kubectl get settings.longhorn.io storage-over-provisioning-percentage -n longhorn-system -o yaml
```

**Output**:

```yaml
apiVersion: longhorn.io/v1beta2
kind: Setting
metadata:
  annotations:
    longhorn.io/configmap-resource-version: "1072"
  creationTimestamp: "2025-09-09T08:20:59Z"
  generation: 1
  name: storage-over-provisioning-percentage
  namespace: longhorn-system
  resourceVersion: "1406"
  uid: c319d1a5-aa00-4ac1-ae8b-d932ecba9073
status:
  applied: true
value: "100"
```

In the output, look for the `value` field to see the current setting.

**Command** (Update the Setting):

To change the value to `500`, run the following `kubectl patch` command:

```bash
kubectl patch settings.longhorn.io storage-over-provisioning-percentage -n longhorn-system --type='json' -p='[{"op": "replace", "path": "/value", "value": "500"}]'
```

**Output**:

```bash
setting.longhorn.io/storage-over-provisioning-percentage patched
```

You can then run the previous `kubectl get` command to verify that the value has been updated.

**Command**:

```bash
kubectl get settings.longhorn.io storage-over-provisioning-percentage -n longhorn-system -o yaml
```

**Output**:

```yaml
apiVersion: longhorn.io/v1beta2
kind: Setting
metadata:
  annotations:
    longhorn.io/configmap-resource-version: "1072"
  creationTimestamp: "2025-09-09T08:20:59Z"
  generation: 2
  name: storage-over-provisioning-percentage
  namespace: longhorn-system
  resourceVersion: "8781"
  uid: c319d1a5-aa00-4ac1-ae8b-d932ecba9073
status:
  applied: true
value: "500"
```

---

## Volume Management with CLI

The following sections will guide you through creating, listing, inspecting, and deleting Longhorn volumes using `kubectl` and Kubernetes Custom Resources (CRs). This method is ideal for automation, scripting, and GitOps workflows.

### 1. Creating a New Volume

In Kubernetes, volumes are provisioned dynamically by a `StorageClass` via a `PersistentVolumeClaim` (PVC). A `PersistentVolume` (PV) is then automatically created. You will create a YAML file for the PVC and apply it.

**Step 1** - Create the PVC Manifest:

Create a file named `longhorn-pvc.yaml` with the following content. This manifest requests a 5GB volume from Longhorn's default `StorageClass`.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: longhorn-test-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: longhorn
  resources:
    requests:
      storage: 5Gi
```

**Step 2** - Apply the Manifest:

Run the following command to create the PVC:

```bash
kubectl apply -f longhorn-pvc.yaml
```

**Output**:

```bash
persistentvolumeclaim/longhorn-test-pvc created
```

### 2. Listing All Volumes

You can view the status of all Longhorn volumes by listing the `PersistentVolumeClaim` (PVCs) and `PersistentVolume` (PVs) in your cluster.

Run the following command to list all PVCs in the current namespace:

```bash
kubectl get pvc
```

**Output**:

```bash
NAME                STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
longhorn-test-pvc   Bound    pvc-eac2af5a-94bd-45e4-a95e-9c3ac7c3b4ff   5Gi        RWO            longhorn       <unset>                 30s
```

You can see the Longhorn PVs that are bound to the PVCs with this command:

```bash
kubectl get pv
```

**Output**:

```bash
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                       STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pvc-eac2af5a-94bd-45e4-a95e-9c3ac7c3b4ff   5Gi        RWO            Delete           Bound    default/longhorn-test-pvc   longhorn       <unset>                          54s
```

### 3. Getting Details of a Specific Volume

To get detailed information about a specific volume, you can inspect its PVC and its corresponding Longhorn volume CR.

Run the following command to get the full YAML output of your new PVC:

```bash
kubectl get pvc longhorn-test-pvc -o yaml
```

**Output**:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"PersistentVolumeClaim","metadata":{"annotations":{},"name":"longhorn-test-pvc","namespace":"default"},"spec":{"accessModes":["ReadWriteOnce"],"resources":{"requests":{"storage":"5Gi"}},"storageClassName":"longhorn"}}
    pv.kubernetes.io/bind-completed: "yes"
    pv.kubernetes.io/bound-by-controller: "yes"
    volume.beta.kubernetes.io/storage-provisioner: driver.longhorn.io
    volume.kubernetes.io/storage-provisioner: driver.longhorn.io
  creationTimestamp: "2025-09-15T13:18:55Z"
  finalizers:
  - kubernetes.io/pvc-protection
  name: longhorn-test-pvc
  namespace: default
  resourceVersion: "15113"
  uid: eac2af5a-94bd-45e4-a95e-9c3ac7c3b4ff
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: longhorn
  volumeMode: Filesystem
  volumeName: pvc-eac2af5a-94bd-45e4-a95e-9c3ac7c3b4ff
status:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 5Gi
  phase: Bound
```

Longhorn also has its own CR for volumes. The name of the Longhorn volume CR will be the same as the name of the `PersistentVolume`. You can find the name of the PV by running `kubectl get pv`. Once you have the PV name, you can inspect the Longhorn volume CR for a deep dive into its status, replicas, and more.

**Command**

Replace `<longhorn-volume-name>` with the name of the PV you found.

```bash
kubectl get volumes.longhorn.io <longhorn-volume-name> -n longhorn-system -o yaml
```

**Output**:

```yaml
apiVersion: longhorn.io/v1beta2
kind: Volume
metadata:
  creationTimestamp: "2025-09-15T13:18:55Z"
  finalizers:
  - longhorn.io
  generation: 1
  labels:
    backup-target: default
    longhornvolume: pvc-eac2af5a-94bd-45e4-a95e-9c3ac7c3b4ff
    recurring-job-group.longhorn.io/default: enabled
    setting.longhorn.io/remove-snapshots-during-filesystem-trim: ignored
  name: pvc-eac2af5a-94bd-45e4-a95e-9c3ac7c3b4ff
  namespace: longhorn-system
  resourceVersion: "15110"
  uid: 103f305e-8444-4b72-b022-f963abcf4c04
spec:
  Standby: false
  accessMode: rwo
  backingImage: ""
  backupCompressionMethod: lz4
  backupTargetName: default
  dataEngine: v1
  dataLocality: disabled
  dataSource: ""
  disableFrontend: false
  diskSelector: []
  encrypted: false
  freezeFilesystemForSnapshot: ignored
  fromBackup: ""
  frontend: blockdev
  image: longhornio/longhorn-engine:v1.9.1
  lastAttachedBy: ""
  migratable: false
  migrationNodeID: ""
  nodeID: ""
  nodeSelector: []
  numberOfReplicas: 3
  offlineRebuilding: ignored
  replicaAutoBalance: ignored
  replicaDiskSoftAntiAffinity: ignored
  replicaSoftAntiAffinity: ignored
  replicaZoneSoftAntiAffinity: ignored
  restoreVolumeRecurringJob: ignored
  revisionCounterDisabled: true
  size: "5368709120"
  snapshotDataIntegrity: ignored
  snapshotMaxCount: 250
  snapshotMaxSize: "0"
  staleReplicaTimeout: 30
  unmapMarkSnapChainRemoved: ignored
status:
  actualSize: 0
  cloneStatus:
    attemptCount: 0
    nextAllowedAttemptAt: ""
    snapshot: ""
    sourceVolume: ""
    state: ""
  conditions:
  - lastProbeTime: ""
    lastTransitionTime: "2025-09-15T13:18:56Z"
    message: ""
    reason: ""
    status: "False"
    type: WaitForBackingImage
  - lastProbeTime: ""
    lastTransitionTime: "2025-09-15T13:18:56Z"
    message: ""
    reason: ""
    status: "False"
    type: TooManySnapshots
  - lastProbeTime: ""
    lastTransitionTime: "2025-09-15T13:18:56Z"
    message: Reset schedulable due to allow volume creation with degraded availability
    reason: ""
    status: "True"
    type: Scheduled
  - lastProbeTime: ""
    lastTransitionTime: "2025-09-15T13:18:56Z"
    message: ""
    reason: ""
    status: "False"
    type: Restore
  currentImage: longhornio/longhorn-engine:v1.9.1
  currentMigrationNodeID: ""
  currentNodeID: ""
  expansionRequired: false
  frontendDisabled: false
  isStandby: false
  kubernetesStatus:
    lastPVCRefAt: ""
    lastPodRefAt: ""
    namespace: default
    pvName: pvc-eac2af5a-94bd-45e4-a95e-9c3ac7c3b4ff
    pvStatus: Bound
    pvcName: longhorn-test-pvc
    workloadsStatus: null
  lastBackup: ""
  lastBackupAt: ""
  lastDegradedAt: ""
  ownerID: ubuntu-lh-2
  remountRequestedAt: ""
  restoreInitiated: false
  restoreRequired: false
  robustness: unknown
  shareEndpoint: ""
  shareState: ""
  state: detached
```

### 4. Deleting a Volume

To delete a volume, you simply need to delete the `PersistentVolumeClaim` (PVC) that created it. This will automatically trigger Longhorn to delete the corresponding PV and the volume data.

Run the following command to delete the PVC:

```bash
kubectl delete pvc longhorn-test-pvc
```

**Output**:

```bash
persistentvolumeclaim "longhorn-test-pvc" deleted
```

---

## Snapshot Management with CLI

The following sections will guide you through creating, listing, inspecting, and deleting Longhorn snapshots using kubectl and Kubernetes Custom Resources (CRs). This method is ideal for automation, scripting, and GitOps workflows.

> **Note**: To run the commands in the Snapshot Management section, you need to first re-create the volume.

### 1. Creating a Snapshot

In the Longhorn UI, a snapshot is created with a single click. Using `kubectl`, you create a snapshot by applying a `VolumeSnapshot` CR. You will need to know the name of the `PersistentVolumeClaim` (PVC) you want to snapshot.

**Step 1** - Create the Snapshot Manifest:

Create a file named `longhorn-snapshot.yaml` with the following content. This manifest creates a snapshot for the `longhorn-test-pvc` you created previously.

```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: test-pvc-snapshot
spec:
  volumeSnapshotClassName: longhorn
  source:
    persistentVolumeClaimName: longhorn-test-pvc
```

**Step 2** - Apply the Manifest:

Run the following command to create the snapshot:

```bash
kubectl apply -f longhorn-snapshot.yaml
```

**Output**:

```bash
volumesnapshot.snapshot.storage.k8s.io/test-pvc-snapshot created
```

### 2. Listing All Snapshots for a Volume

To see the snapshots that have been created, you can list the `volumesnapshots` CRs in your cluster.

Run the following command to list all snapshots:

```bash
kubectl get volumesnapshot
```

**Output**:

```bash
NAME                READYTOUSE   SOURCEPVC           SOURCESNAPSHOTCONTENT   RESTORESIZE   SNAPSHOTCLASS   SNAPSHOTCONTENT   CREATIONTIME   AGE
test-pvc-snapshot                longhorn-test-pvc                                         longhorn                                         31s
```

### 3. Restoring a Volume from a Snapshot

Restoring a volume from a snapshot involves creating a new PVC with the snapshot as its data source. You must first delete the old PVC before you can restore the volume.

**Step 1** - Delete the Original PVC:

If the original PVC still exists, you must delete it first.

```bash
kubectl delete pvc longhorn-test-pvc
```

**Output**:

```bash
persistentvolumeclaim "longhorn-test-pvc" deleted
```

**Step 2** - Create a PVC from the Snapshot:

Create a new manifest file, `longhorn-pvc-restore.yaml`, to create a new PVC that uses the snapshot as its source.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: restored-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: longhorn
  dataSource:
    name: test-pvc-snapshot
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
  resources:
    requests:
      storage: 5Gi
```

**Step 3** - Apply the Manifest:

Run the following command to create the new PVC:

```bash
kubectl apply -f longhorn-pvc-restore.yaml
```

**Output**:

```bash
persistentvolumeclaim/restored-pvc created
```

You can then check the status of the new PVC with `kubectl get pvc`.

### 4. Deleting a Snapshot

To delete a snapshot, you simply need to delete the `VolumeSnapshot` CR.

Run the following command to delete the snapshot:

```bash
kubectl delete volumesnapshot test-pvc-snapshot
```

**Output**:

```bash
volumesnapshot.snapshot.storage.k8s.io "test-pvc-snapshot" deleted
```

---

## Recurring Jobs with CLI

The following sections will guide you through creating, listing, inspecting, and deleting recurring jobs using `kubectl` and Kubernetes Custom Resources (CRs). Recurring jobs are essential for automating your backup and snapshot policies.

### 1. Creating a Recurring Job

In the Longhorn UI, you can create a recurring job with a few clicks. Using `kubectl`, you will create a `RecurringJob` CR. This job will automatically take snapshots of a volume on a set schedule.

**Step 1** - Create the Recurring Job Manifest:

Create a file named `longhorn-recurring-job.yaml` with the following content. This manifest creates a job that takes a snapshot every 1440 minutes (24 hours) and keeps 5 snapshots. It will apply to any volume with the label `longhorn.io/recurring-job-group: default`.

```yaml
apiVersion: longhorn.io/v1beta2
kind: RecurringJob
metadata:
  name: default-snapshot
  namespace: longhorn-system
spec:
  cron: "0 0 * * *"
  task: "snapshot"
  retain: 5
  groups:
  - default
```

**Step 2** - Label Your Volume:

You must label the `PersistentVolumeClaim` (PVC) you want the job to apply to. This is the `longhorn-test-pvc` you created previously.

```bash
kubectl label pvc longhorn-test-pvc longhorn.io/recurring-job-group=default
```

**Output**:

```bash
persistentvolumeclaim/longhorn-test-pvc labeled
```

**Step 3** - Apply the Manifest:

Run the following command to create the recurring job:

```bash
kubectl apply -f longhorn-recurring-job.yaml
```

**Output**:

```bash
recurringjob.longhorn.io/default-snapshot created
```

### 2. Listing and Inspecting Job Details

You can view the status of all recurring jobs and their details by listing the `recurringjobs` CRs in the `longhorn-system` namespace.

Run the following command to list all recurring jobs:

```bash
kubectl get recurringjobs -n longhorn-system
```

**Output**:

```bash
NAME               GROUPS        TASK       CRON        RETAIN   CONCURRENCY   AGE   LABELS
default-snapshot   ["default"]   snapshot   0 0 * * *   5                      22s   {}
```

You can then get a full YAML view of a specific job to inspect its configuration and status. Run the following command to get the full YAML output of your new job:

```bash
kubectl get recurringjobs default-snapshot -n longhorn-system -o yaml
```

**Output**:

```yaml
apiVersion: longhorn.io/v1beta2
kind: RecurringJob
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"longhorn.io/v1beta2","kind":"RecurringJob","metadata":{"annotations":{},"name":"default-snapshot","namespace":"longhorn-system"},"spec":{"cron":"0 0 * * *","groups":["default"],"retain":5,"task":"snapshot"}}
  creationTimestamp: "2025-09-15T14:12:13Z"
  generation: 1
  name: default-snapshot
  namespace: longhorn-system
  resourceVersion: "19446"
  uid: b42af778-8436-481e-a405-0f330f0b7115
spec:
  cron: 0 0 * * *
  groups:
  - default
  labels: {}
  name: default-snapshot
  parameters: {}
  retain: 5
  task: snapshot
status:
  executionCount: 0
  ownerID: ubuntu-lh-2
```

### 3. Deleting a Recurring Job

To delete a recurring job, you simply need to delete the `RecurringJob` CR. This will stop the automatic snapshots from being taken.

**Command**

Run the following command to delete the recurring job:

```bash
kubectl delete recurringjobs default-snapshot -n longhorn-system
```

**Output**:

```bash
recurringjob.longhorn.io "default-snapshot" deleted
```
