# Coverage: Where Every Original Note Went

This file is the **proof that nothing was lost** when the 59 `Day NN/` folders were reorganised into the 89 topics listed in [INDEX.md](INDEX.md). The `Day NN/` folders are **not included** in this repo. To see an original file, open it in the original course repo: [CloudWithVarJosh/CKA-Certification-Course-2025](https://github.com/CloudWithVarJosh/CKA-Certification-Course-2025) (e.g. `Day 12/frontend.yaml`). `images/` is kept and unchanged.

## How to read this

- There is one section per original Day. Each section has up to three tables:
  1. **Original section → New home.** Every level-2 (`##`) and level-3 (`###`, shown with ↳) heading of the Day's `README.md`, with the topic(s) that now contain that content (rewritten, expanded, and corrected where needed). Headings that appear only inside code blocks are ignored.
  2. **Image → Used in.** Every image the Day's README referenced, and the topic(s) that now display it.
  3. **File → Used in.** Every other file in the Day folder (YAML manifests, Dockerfiles, app code, charts, kind configs…) and the topic(s) that walk through it.
- When a section lists **two topics**, the first is its main home and the second also uses part of it (e.g. Day 07's Python→Redis flow is explained in topic 32 and its CNI part in topic 29).
- Three kinds of row are **not carried over** as content, and are marked as such: the YouTube "Video reference" links, the "⭐ Support the Project" star requests and the "Table of Contents" blocks. They are navigation only, and still in the original folders in the [original course repo](https://github.com/CloudWithVarJosh/CKA-Certification-Course-2025). Each topic's **Appendix → Sources** section names the Days and sections it was built from, as a cross-check in the other direction.
- **Lecture links and references:** links to other Days' videos/notes ("revisit Day 12…") became cross-references to the matching topic numbers. External documentation links were kept where they add value.

## Images not referenced by any original note

Two images in `images/` weren't referenced by any original README. Both are now used:

| Image | Content | Now used in |
|---|---|---|
| `12i.png` | LoadBalancer Service flow for `frontend-svc` | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md) |
| `31-11.png` | Seema's kubectl ↔ API server TLS with the Kubernetes CA | [51](notes/51-certificates-and-cas.md) |

(`images/test` is a 2-byte placeholder text file, not an image.)

## Corrections index

Wherever the original notes were wrong or outdated, the original statement is kept in full in the topic's **Appendix → Corrections to the original notes** as **Cn · Corrected (Day NN).** "what the original said". **Now:** what's true, and why. In the topic body, the current fact appears as a short ✅ **Correct today** line, or as a ⚠️ **Fix to the original** box when the whole explanation is an instruction you'll follow. Either way it links to its Cn entry. Smaller precision points are ℹ️ **Note** boxes in the body. There are about **128 corrections** across **58 topics**, plus about **24 clarifications**. Search any topic for `Corrected (Day` to find its appendix entries.

| Topic | Corrections | Topic | Corrections | Topic | Corrections |
|---|---|---|---|---|---|
| 01 | 2 | 22 | 3 | 49 | 2 |
| 02 | 1 | 23 | 3 | 50 | 3 |
| 03 | 2 | 25 | 1 | 51 | 1 |
| 05 | 1 | 26 | 2 | 52 | 1 |
| 06 | 3 | 28 | 4 | 53 | 2 |
| 08 | 1 | 30 | 3 | 54 | 4 |
| 09 | 2 | 31 | 3 | 55 | 3 |
| 10 | 4 | 32 | 4 | 56 | 3 |
| 11 | 1 | 33 | 4 | 57 | 3 |
| 12 | 1 | 36 | 1 | 58 | 4 |
| 13 | 2 | 37 | 1 | 59 | 3 |
| 14 | 2 | 38 | 5 | 61 | 2 |
| 15 | 1 | 40 | 1 | 62 | 3 |
| 16 | 1 | 41 | 2 | 64 | 2 |
| 19 | 2 | 42 | 1 | 65 | 5 |
| 20 | 1 | 43 | 1 | 71–73 | 1 / 1 / 2 |
| 21 | 1 | 47 | 3 | 75–76 | 3 / 3 |
|  |  | 48 | 2 | 79 / 82 / 83 | 1 / 2 / 1 |

## New topics (no single original Day)

These topics were written to complete the path to production. Each one names, in its appendix, which original Days it builds on: [04](notes/04-linux-under-kubernetes.md), [29](notes/29-network-model-and-cni.md), [32](notes/32-inside-a-service-kube-proxy.md), [46](notes/46-pod-affinity-and-topology-spread.md), [60](notes/60-pod-security-seccomp-apparmor.md), [63](notes/63-supply-chain-and-secrets-in-production.md), [68](notes/68-gitops.md), [74](notes/74-service-mesh-essentials.md), [77](notes/77-etcd-backup-restore-and-dr.md), [78](notes/78-ha-control-plane-and-managed-kubernetes.md), [80](notes/80-production-observability.md), [81](notes/81-troubleshooting-method.md), [84](notes/84-production-break-fix-scenarios.md), [85](notes/85-production-readiness-checklist.md), [86](notes/86-interview-question-bank.md), [87](notes/87-senior-scenario-interviews.md), [88](notes/88-cka-exam-strategy.md), [89](notes/89-cheat-sheet.md).

---

## Masterclass folders (Storage and TLS)

Two extra folders, `Storage-In-Kubernetes-Masterclass/` and `TLS-In-Kubernetes-Masterclass/`, were checked against these notes on 2026-11-19. Each is a single `README.md` that **merges course Days**: Storage = Days 24–28, TLS = Days 30–34. A line-by-line comparison against the original Day READMEs found:

- **Identical content** everywhere except the intro/conclusion summaries, the tables of contents, one reference list (Storage) and one rewritten paragraph (TLS, Example 3 Step 3).
- **All 35 images** are byte-identical to files in `images/`. **All 17 YAML files** are byte-identical to the original `Day 26`/`Day 27`/`Day 28` files, whose content is written out in the topics listed below.

What was merged from them:

| Item | Label | Where it lives now |
|---|---|---|
| Storage masterclass reference list (19 official Docker/Kubernetes doc links) | ➕ More detail | **Official references** in the appendix of [09](notes/09-plugins-interfaces-and-add-ons.md), [34](notes/34-configmaps.md), [35](notes/35-secrets.md), [36](notes/36-docker-storage-recap.md), [37](notes/37-ephemeral-volumes.md), [38](notes/38-persistent-storage-pv-pvc.md), [39](notes/39-storageclasses-dynamic-provisioning.md) |
| Storage: "Recycle (Deprecated in v1.20+) … It was removed" | ⚠️ Conflict | Corrected in [38](notes/38-persistent-storage-pv-pvc.md#c5) (C5) |
| TLS: Example 3 Step 3 rewrite ("trust is enforced at the client-auth side"; API server "relies on RBAC, Node authorizer, and NodeRestriction") | ⚠️ Conflict | Corrected in [53](notes/53-tls-inside-the-cluster.md#c2) (C2) |
| A stray AI-chat line in the Storage masterclass ("Sure! Here's a refined and enhanced version…") | Not content | Not carried over |

Every heading is listed below (`↳` = level 3, `↳↳` = level 4; headings inside code blocks are ignored).

### Storage-In-Kubernetes-Masterclass · README.md

| Masterclass section | New home |
|---|---|
| Storage in Kubernetes MASTERCLASS | Not carried over (video link / repo star request / table of contents: navigation only) |
| Video reference for the MASTERCLASS is the following: | Not carried over (video link / repo star request / table of contents: navigation only) |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only) |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only) |
| ↳ Introduction | Not carried over as content: a summary of the course flow, which the learning path in [INDEX.md](INDEX.md) already gives |
| ↳ Understanding Storage in Docker | [36](notes/36-docker-storage-recap.md) |
| ↳ Layers in a Dockerfile | [36](notes/36-docker-storage-recap.md) |
| ↳ Example: Layers in Our Dockerfile | [36](notes/36-docker-storage-recap.md) |
| ↳ How to Inspect Image Layers | [36](notes/36-docker-storage-recap.md) |
| ↳ How Docker Containers Use Image Layers | [36](notes/36-docker-storage-recap.md) |
| ↳ Storage Calculation Example | [36](notes/36-docker-storage-recap.md) |
| ↳ Handling Data Loss in Writable Layers | [36](notes/36-docker-storage-recap.md) |
| ↳ Need for Persistent Storage | [36](notes/36-docker-storage-recap.md) |
| Storage in Docker: How Does It Work? | [36](notes/36-docker-storage-recap.md) |
| ↳ 1. Storage Drivers | [36](notes/36-docker-storage-recap.md) |
| ↳ 2. Volume Drivers | [36](notes/36-docker-storage-recap.md) |
| How Docker Storage Works on macOS, Windows, and Linux | [36](notes/36-docker-storage-recap.md) |
| ↳ Linux | [36](notes/36-docker-storage-recap.md) |
| ↳ macOS and Windows | [36](notes/36-docker-storage-recap.md) |
| ↳ Understanding Docker Volumes | [36](notes/36-docker-storage-recap.md) |
| ↳ Where Are Docker Volumes Stored? | [36](notes/36-docker-storage-recap.md) |
| Demo: With Persistent Storage (Docker Volume) | [36](notes/36-docker-storage-recap.md) |
| ↳ Note for macOS and Windows Users | [36](notes/36-docker-storage-recap.md) |
| ↳↳ How to Access Volume Data on macOS: | [36](notes/36-docker-storage-recap.md) |
| ↳ Types of Docker Volumes: Which to Use When? | [36](notes/36-docker-storage-recap.md) |
| Kubernetes Core and Extended Architecture | [06](notes/06-the-cluster-at-a-glance.md) |
| Kubernetes Core Architecture | [06](notes/06-the-cluster-at-a-glance.md) |
| ↳ What is Kubernetes Core? | [06](notes/06-the-cluster-at-a-glance.md) |
| ↳ 1️⃣ Control Plane Components | [07](notes/07-control-plane-components.md) |
| ↳ 2️⃣ Node Components | [08](notes/08-node-components.md) |
| Kubernetes Extensions: Beyond the Core | [09](notes/09-plugins-interfaces-and-add-ons.md) |
| ↳ 1️⃣ Plugins | [09](notes/09-plugins-interfaces-and-add-ons.md) |
| ↳↳ CRI Plugins (Container Runtimes) | [09](notes/09-plugins-interfaces-and-add-ons.md) |
| ↳↳ CNI Plugins (Networking) | [09](notes/09-plugins-interfaces-and-add-ons.md) |
| ↳↳ CSI Plugins (Storage) | [09](notes/09-plugins-interfaces-and-add-ons.md) |
| ↳↳ Responsibilities of CRI, CNI, and CSI in Kubernetes | [09](notes/09-plugins-interfaces-and-add-ons.md) |
| ↳ 2️⃣ Add-Ons | [09](notes/09-plugins-interfaces-and-add-ons.md) |
| ↳ 3️⃣ Third-Party Extensions | [09](notes/09-plugins-interfaces-and-add-ons.md) |
| Deep Dive into Kubernetes Interfaces | [06](notes/06-the-cluster-at-a-glance.md) [09](notes/09-plugins-interfaces-and-add-ons.md) |
| ↳ Overview of Kubernetes Interfaces | [06](notes/06-the-cluster-at-a-glance.md) [09](notes/09-plugins-interfaces-and-add-ons.md) |
| Why Kubernetes Uses a Plugin-Based Architecture? | [09](notes/09-plugins-interfaces-and-add-ons.md) |
| Key Takeaways | [06](notes/06-the-cluster-at-a-glance.md) |
| ↳ Ephemeral Storage | [37](notes/37-ephemeral-volumes.md) |
| ↳↳ EmptyDir | [37](notes/37-ephemeral-volumes.md) |
| ↳↳ Demo: emptyDir | [37](notes/37-ephemeral-volumes.md) |
| ↳↳ Downward API | [37](notes/37-ephemeral-volumes.md) |
| ↳↳ Why is it Used? | [37](notes/37-ephemeral-volumes.md) |
| ↳↳ Demo: downwardAPI - Environment variables | [37](notes/37-ephemeral-volumes.md) |
| ↳↳ Demo: downwardAPI - Files via Volumes | [37](notes/37-ephemeral-volumes.md) |
| ↳↳ ConfigMaps & Secrets (To Be Discussed in Next Lecture) | [37](notes/37-ephemeral-volumes.md) (preview; full topics [34](notes/34-configmaps.md), [35](notes/35-secrets.md)) |
| Evolution of Storage in Kubernetes: From In-Tree to CSI Drivers | [38](notes/38-persistent-storage-pv-pvc.md) |
| 1. In-Tree Volume Plugins: The Legacy Model | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳ Examples of In-Tree Plugins: | [38](notes/38-persistent-storage-pv-pvc.md) |
| 2. Container Storage Interface (CSI): The Modern Standard | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳ Key Benefits of CSI: | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳ Examples of CSI Drivers: | [38](notes/38-persistent-storage-pv-pvc.md) |
| Special Cases: hostPath and local Storage | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳ hostPath | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳ local Volumes | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳↳ Transitioning from In-Tree Drivers | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳↳ Key Takeaways: | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳ Persistent Storage | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳↳ hostPath | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳ Understanding hostPath in KIND: How Storage Works Under the Hood | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳↳ 1. On Ubuntu/Linux | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳↳ 2. On macOS and Windows | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳ Key Takeaway | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳ Why This Matters | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳ Demo: hostPath | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳ Persistent Volumes (PVs) & Persistent Volume Claims (PVCs) | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳↳ Persistent Volumes (PVs) | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳↳ Persistent Volume Claims (PVCs) | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳ Understanding Scope & Relationships of PV and PVC in Kubernetes | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳↳ PVs are Cluster-Scoped Resources | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳↳ PVCs are Namespace-Scoped | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳ Why Is This Important? | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳ 1-to-1 Binding Relationship Between PVC and PV | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳ Additional Key Points | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳ Example Scenario | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳ Kubernetes Persistent Storage Flow (Manual Provisioning) | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳ Important Notes | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳↳ Access Modes in Kubernetes Persistent Volumes | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳ Explanation of Storage Types | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳↳ Block Storage | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳↳ File Storage | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳ Key Differences: Block Storage vs. File Storage | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳ Reclaim Policies in Kubernetes | [38](notes/38-persistent-storage-pv-pvc.md) (Recycle claim corrected: C5) |
| ↳↳ 1. Delete (Common for Dynamically Provisioned Storage) | [38](notes/38-persistent-storage-pv-pvc.md) (Recycle claim corrected: C5) |
| ↳↳ 2. Retain (Manual Intervention Needed for Reuse) | [38](notes/38-persistent-storage-pv-pvc.md) (Recycle claim corrected: C5) |
| ↳↳ 3. Recycle (Deprecated in Kubernetes v1.20+) | [38](notes/38-persistent-storage-pv-pvc.md) (Recycle claim corrected: C5) |
| ↳ Choosing the Right Reclaim Policy | [38](notes/38-persistent-storage-pv-pvc.md) (Recycle claim corrected: C5) |
| ↳ PVC and PV Binding Conditions | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳ Summary Table: PVC and PV Binding Conditions | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳ Example Table: PVC vs. PV Matching | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳ Critical Note for KIND/Minikube Users | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳↳ Why Modify the Default Storage Class? | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳↳ Steps to Back Up and Delete the Storage Class | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳ Demo: Persistent Volumes and PVCs with Reclaim Policy | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳↳ Step 1: Create a Persistent Volume (PV) | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳↳ Step 2: Create a Persistent Volume Claim (PVC) | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳↳ Step 3: Create a Pod That Uses the PVC | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳↳ Final Verification | [38](notes/38-persistent-storage-pv-pvc.md) |
| ↳ Storage Classes & Dynamic Provisioning | [39](notes/39-storageclasses-dynamic-provisioning.md) |
| ↳↳ What is a Storage Class? | [39](notes/39-storageclasses-dynamic-provisioning.md) |
| ↳↳ Why Is a Storage Class Required? | [39](notes/39-storageclasses-dynamic-provisioning.md) |
| ↳ Example Storage Classes | [39](notes/39-storageclasses-dynamic-provisioning.md) |
| ↳↳ Key Points | [39](notes/39-storageclasses-dynamic-provisioning.md) |
| ↳ Dynamic Provisioning in Action | [39](notes/39-storageclasses-dynamic-provisioning.md) |
| ↳↳ Persistent Volume Claim (PVC) | [39](notes/39-storageclasses-dynamic-provisioning.md) |
| ↳↳ Persistent Volume (PV) | [39](notes/39-storageclasses-dynamic-provisioning.md) |
| ↳↳ Pod Using PVC | [39](notes/39-storageclasses-dynamic-provisioning.md) |
| ↳↳ Key Takeaways | [39](notes/39-storageclasses-dynamic-provisioning.md) |
| ↳ Demo: Storage Class | [39](notes/39-storageclasses-dynamic-provisioning.md) |
| ↳↳ Step 1: Reapply the Storage Class | [39](notes/39-storageclasses-dynamic-provisioning.md) |
| ↳↳ Step 2: Create the PersistentVolumeClaim (PVC) | [39](notes/39-storageclasses-dynamic-provisioning.md) |
| ↳↳ Step 3: Understand VolumeBindingMode | [39](notes/39-storageclasses-dynamic-provisioning.md) |
| ↳↳ Step 4: Create a Pod Using the PVC | [39](notes/39-storageclasses-dynamic-provisioning.md) |
| ↳↳ Step 5: Verify the Status | [39](notes/39-storageclasses-dynamic-provisioning.md) |
| ↳ Key Takeaways | [39](notes/39-storageclasses-dynamic-provisioning.md) |
| ConfigMaps | [34](notes/34-configmaps.md) |
| ↳ Why Do We Need ConfigMaps? | [34](notes/34-configmaps.md) |
| ↳ What is a ConfigMap? | [34](notes/34-configmaps.md) |
| ↳ Without ConfigMap: Hardcoded Environment Variables | [34](notes/34-configmaps.md) |
| Demo 1: Using ConfigMap as Environment Variables | [34](notes/34-configmaps.md) |
| ↳ Step 1: Create ConfigMap | [34](notes/34-configmaps.md) |
| ↳ Step 2: Reference ConfigMap in Deployment | [34](notes/34-configmaps.md) |
| ↳ Verification | [34](notes/34-configmaps.md) |
| Demo 2: Using ConfigMap as a Configuration File (Volume Mount) | [34](notes/34-configmaps.md) |
| ↳ Step 1: Modify ConfigMap to Include HTML File | [34](notes/34-configmaps.md) |
| ↳ Step 2: Mount ConfigMap Volume in Deployment | [34](notes/34-configmaps.md) |
| ↳ What if you *do* want to mount the ConfigMap as a single file? | [34](notes/34-configmaps.md) |
| Step 3: Expose the Deployment using a NodePort Service | [34](notes/34-configmaps.md) |
| ↳ NodePort Service Manifest | [34](notes/34-configmaps.md) |
| Verification | [34](notes/34-configmaps.md) |
| ↳↳ 1. Get the Pod name | [34](notes/34-configmaps.md) |
| ↳↳ 2. Verify the index.html content inside the container | [34](notes/34-configmaps.md) |
| ↳↳ 3. Access the Application via NodePort | [34](notes/34-configmaps.md) |
| ↳↳ 4. If You're Using KIND (Kubernetes in Docker) | [34](notes/34-configmaps.md) |
| ↳ Best Practices for ConfigMap | [34](notes/34-configmaps.md) |
| Kubernetes Secrets | [35](notes/35-secrets.md) |
| ↳ Why Do We Need Secrets in Kubernetes? | [35](notes/35-secrets.md) |
| ↳ What is a Kubernetes Secret? | [35](notes/35-secrets.md) |
| ↳ Important Distinction: Encoding vs. Encryption | [35](notes/35-secrets.md) |
| ↳ Encoding vs. Encryption | [35](notes/35-secrets.md) |
| ↳ Demo 1: Injecting Secrets into a Pod | [35](notes/35-secrets.md) |
| ↳ Step 1: Create the Secret | [35](notes/35-secrets.md) |
| ↳ Step 2: Update the Deployment to Use the Secret | [35](notes/35-secrets.md) |
| ↳ Step 3: Verify Secret Injection | [35](notes/35-secrets.md) |
| ↳ Alternative: Mount Secret as Files | [35](notes/35-secrets.md) |
| ↳↳ Update container spec: | [35](notes/35-secrets.md) |
| ↳↳ Add volume to spec: | [35](notes/35-secrets.md) |
| ↳ Best Practices for Using Kubernetes Secrets | [35](notes/35-secrets.md) |
| ↳ Understanding Dynamic Updates with ConfigMaps and Secrets | [35](notes/35-secrets.md) |
| ↳ Conclusion | Not carried over as content: a summary of the course flow, which the learning path in [INDEX.md](INDEX.md) already gives |
| ↳ References | Official references in [09](notes/09-plugins-interfaces-and-add-ons.md), [34](notes/34-configmaps.md), [35](notes/35-secrets.md), [36](notes/36-docker-storage-recap.md), [37](notes/37-ephemeral-volumes.md), [38](notes/38-persistent-storage-pv-pvc.md), [39](notes/39-storageclasses-dynamic-provisioning.md) |

| Image | Used in |
|---|---|
| `24a.png` | [36](notes/36-docker-storage-recap.md)  |
| `24b.png` | [36](notes/36-docker-storage-recap.md)  |
| `24c.png` | [36](notes/36-docker-storage-recap.md)  |
| `24d.png` | [36](notes/36-docker-storage-recap.md)  |
| `24e.png` | [36](notes/36-docker-storage-recap.md)  |
| `25a.png` | [06](notes/06-the-cluster-at-a-glance.md)  |
| `25b.png` | [09](notes/09-plugins-interfaces-and-add-ons.md)  |
| `26a.png` | [37](notes/37-ephemeral-volumes.md)  |
| `27a.png` | [38](notes/38-persistent-storage-pv-pvc.md)  |
| `27b.png` | [38](notes/38-persistent-storage-pv-pvc.md)  |
| `27c.png` | [38](notes/38-persistent-storage-pv-pvc.md)  |
| `27d.png` | [39](notes/39-storageclasses-dynamic-provisioning.md)  |
| `27e.png` | [38](notes/38-persistent-storage-pv-pvc.md)  |

| File | Used in |
|---|---|
| `backend-2.yaml` | [35](notes/35-secrets.md)  |
| `cm-1.yaml` | [34](notes/34-configmaps.md)  |
| `cm-2.yaml` | [34](notes/34-configmaps.md)  |
| `downwardapi-1.yaml` | [37](notes/37-ephemeral-volumes.md)  |
| `downwardapi-2.yaml` | [37](notes/37-ephemeral-volumes.md)  |
| `emptyDir.yaml` | [37](notes/37-ephemeral-volumes.md)  |
| `ev.yaml` | [34](notes/34-configmaps.md)  |
| `frontend-cm-1.yaml` | [34](notes/34-configmaps.md)  |
| `frontend-cm-2.yaml` | [34](notes/34-configmaps.md)  |
| `frontend-secret-1.yaml` | [35](notes/35-secrets.md)  |
| `frontend-secret-2.yaml` | [35](notes/35-secrets.md)  |
| `hostpath.yaml` | [38](notes/38-persistent-storage-pv-pvc.md)  |
| `pod.yaml` | [13](notes/13-yaml-and-manifest-anatomy.md) [18](notes/18-pods.md)  |
| `pv.yaml` | [38](notes/38-persistent-storage-pv-pvc.md)  |
| `pvc.yaml` | [38](notes/38-persistent-storage-pv-pvc.md)  |
| `sc.yaml` | [38](notes/38-persistent-storage-pv-pvc.md) [39](notes/39-storageclasses-dynamic-provisioning.md)  |
| `secret-1.yaml` | [35](notes/35-secrets.md)  |

### TLS-In-Kubernetes-Masterclass · README.md

| Masterclass section | New home |
|---|---|
| TLS in Kubernetes MASTERCLASS \| mTLS, CSR, CA, SSH, kubeconfig & Cluster Security Explained | Not carried over (video link / repo star request / table of contents: navigation only) |
| Video reference for the Masterclass is the following: | Not carried over (video link / repo star request / table of contents: navigation only) |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only) |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only) |
| TLS in Kubernetes MASTERCLASS — Full Series | Not carried over as content: a summary of the course flow, which the learning path in [INDEX.md](INDEX.md) already gives |
| Introduction | Not carried over as content: a summary of the course flow, which the learning path in [INDEX.md](INDEX.md) already gives |
| What is Encryption? Why Do We Need It? | [50](notes/50-encryption-basics.md) |
| Two Main Types of Encryption | [50](notes/50-encryption-basics.md) |
| ↳ 1. Symmetric Encryption (Shared Key Encryption) | [50](notes/50-encryption-basics.md) |
| ↳ 2. Asymmetric Encryption (Public Key Cryptography) | [50](notes/50-encryption-basics.md) |
| Summary Table | [50](notes/50-encryption-basics.md) |
| ↳ Symmetric and Asymmetric Encryption: Roles and Functions | [50](notes/50-encryption-basics.md) |
| Encryption Approaches: In-Transit vs At-Rest | [50](notes/50-encryption-basics.md) |
| ↳ 1. Encryption In-Transit | [50](notes/50-encryption-basics.md) |
| ↳ 2. Encryption At-Rest | [50](notes/50-encryption-basics.md) |
| ↳ Quick Comparison: | [50](notes/50-encryption-basics.md) |
| Understanding Symmetric and Asymmetric Encryption Through Real-Life Scenarios | [50](notes/50-encryption-basics.md) |
| ↳ Scenario 1: Symmetric Encryption (Disk Encryption) | [50](notes/50-encryption-basics.md) |
| ↳ Scenario 2: Symmetric Encryption Alone Isn't Enough for Internet Communication | [50](notes/50-encryption-basics.md) |
| ↳ Scenario 3: Asymmetric Encryption (SSH into EC2 Instance) | [50](notes/50-encryption-basics.md) |
| ↳↳ Steps: | [50](notes/50-encryption-basics.md) |
| ↳↳ Key Concepts: | [50](notes/50-encryption-basics.md) |
| ↳↳ How SSH Uses Asymmetric Encryption: | [50](notes/50-encryption-basics.md) |
| ↳ Scenario 4: Combining Symmetric + Asymmetric (TLS Handshake) | [50](notes/50-encryption-basics.md) |
| ↳ Client and Server – A Refresher | [51](notes/51-certificates-and-cas.md) |
| Public Key Cryptography | [51](notes/51-certificates-and-cas.md) |
| ↳ Public Key Cryptography in DevOps: Focus on SSH & HTTPS | [51](notes/51-certificates-and-cas.md) |
| ↳↳ 1. Secure Remote Access: ssh-keygen (SSH Authentication) | [51](notes/51-certificates-and-cas.md) |
| ↳↳ 2. Secure Web Communication: openssl (TLS Certificates & Identity Validation) | [51](notes/51-certificates-and-cas.md) |
| ↳↳ Key Management Best Practices | [51](notes/51-certificates-and-cas.md) |
| ↳↳ Common Key File Formats | [51](notes/51-certificates-and-cas.md) |
| ↳↳ For SSH Authentication | [51](notes/51-certificates-and-cas.md) |
| ↳↳ For TLS Certificates | [51](notes/51-certificates-and-cas.md) |
| ↳↳ What Public Key Cryptography (PKC) Provides | [51](notes/51-certificates-and-cas.md) |
| ↳ 1. Authentication (Identity Verification): | [51](notes/51-certificates-and-cas.md) |
| ↳↳ In SSH: | [51](notes/51-certificates-and-cas.md) |
| ↳↳ In TLS (e.g., HTTPS): | [51](notes/51-certificates-and-cas.md) |
| ↳ Order of Authentication in Mutual Authentication (SSH or mTLS) | [51](notes/51-certificates-and-cas.md) |
| ↳ 2. Secure Key Exchange (Foundation for Encryption): | [51](notes/51-certificates-and-cas.md) |
| ↳ Types of TLS Certificate Authorities (CA): Public, Private, and Self-Signed | [51](notes/51-certificates-and-cas.md) |
| ↳ Public CA | [51](notes/51-certificates-and-cas.md) |
| ↳ Private CA | [51](notes/51-certificates-and-cas.md) |
| ↳ Self-Signed Certificate | [51](notes/51-certificates-and-cas.md) |
| ↳↳ Typical Use Cases of Self-Signed Certificates | [51](notes/51-certificates-and-cas.md) |
| ↳↳ Common Internal Domain Suffixes for Testing | [51](notes/51-certificates-and-cas.md) |
| Kubeconfig and Kubernetes Context | [52](notes/52-kubeconfig-deep-dive-and-mtls.md) [12](notes/12-kubectl-kubeconfig-and-contexts.md) |
| ↳ What is a Kubeconfig File and Why Do We Need It? | [52](notes/52-kubeconfig-deep-dive-and-mtls.md) [12](notes/12-kubectl-kubeconfig-and-contexts.md) |
| ↳ What If the Kubeconfig File Were Not There? | [52](notes/52-kubeconfig-deep-dive-and-mtls.md) [12](notes/12-kubectl-kubeconfig-and-contexts.md) |
| ↳ Kubernetes Context | [52](notes/52-kubeconfig-deep-dive-and-mtls.md) [12](notes/12-kubectl-kubeconfig-and-contexts.md) |
| Scenario: | [52](notes/52-kubeconfig-deep-dive-and-mtls.md) [12](notes/12-kubectl-kubeconfig-and-contexts.md) |
| How Kubernetes Contexts Work: | [52](notes/52-kubeconfig-deep-dive-and-mtls.md) [12](notes/12-kubectl-kubeconfig-and-contexts.md) |
| ↳ 1. Configuring Contexts: | [52](notes/52-kubeconfig-deep-dive-and-mtls.md) [12](notes/12-kubectl-kubeconfig-and-contexts.md) |
| ↳ 2. Switching Between Contexts: | [52](notes/52-kubeconfig-deep-dive-and-mtls.md) [12](notes/12-kubectl-kubeconfig-and-contexts.md) |
| Example kubeconfig File: | [52](notes/52-kubeconfig-deep-dive-and-mtls.md) [12](notes/12-kubectl-kubeconfig-and-contexts.md) |
| ↳ Viewing & Managing Your kubeconfig | [52](notes/52-kubeconfig-deep-dive-and-mtls.md) [12](notes/12-kubectl-kubeconfig-and-contexts.md) |
| ↳ Common kubectl config Commands (Compact View) | [52](notes/52-kubeconfig-deep-dive-and-mtls.md) [12](notes/12-kubectl-kubeconfig-and-contexts.md) |
| ↳ Multiple Kubeconfig Files | [52](notes/52-kubeconfig-deep-dive-and-mtls.md) [12](notes/12-kubectl-kubeconfig-and-contexts.md) |
| ↳ Using a Custom Kubeconfig as Default via KUBECONFIG | [52](notes/52-kubeconfig-deep-dive-and-mtls.md) [12](notes/12-kubectl-kubeconfig-and-contexts.md) |
| ↳ Adding Entries to a Kubeconfig File | [52](notes/52-kubeconfig-deep-dive-and-mtls.md) [12](notes/12-kubectl-kubeconfig-and-contexts.md) |
| Mutual TLS (mTLS) | [52](notes/52-kubeconfig-deep-dive-and-mtls.md) |
| ↳ Private CAs in Kubernetes Clusters | [53](notes/53-tls-inside-the-cluster.md) |
| ↳ Kubernetes Components as Clients and Servers | [53](notes/53-tls-inside-the-cluster.md) |
| ↳ Client (Initiates a Request) | [53](notes/53-tls-inside-the-cluster.md) |
| ↳ Server (Responds to the Request) | [53](notes/53-tls-inside-the-cluster.md) |
| Understanding TLS in Kubernetes: No More Memorizing File Paths | [53](notes/53-tls-inside-the-cluster.md) |
| Example 1: mTLS Between Scheduler (Client) and API Server (Server) | [53](notes/53-tls-inside-the-cluster.md) |
| ↳ Key Principle: All Clients Use a Kubeconfig File to Connect to the API Server | [53](notes/53-tls-inside-the-cluster.md) |
| Key TLS and Configuration File Locations (Control Plane & Nodes) | [53](notes/53-tls-inside-the-cluster.md) |
| SERVER-SIDE: How the API Server Presents Its Certificate | [53](notes/53-tls-inside-the-cluster.md) |
| ↳ Step 1: Who’s the Server? | [53](notes/53-tls-inside-the-cluster.md) |
| ↳ Step 2: How Does the Server Present a Certificate? | [53](notes/53-tls-inside-the-cluster.md) |
| ↳ Step 3: How Does the Scheduler Trust the API Server’s Certificate? | [53](notes/53-tls-inside-the-cluster.md) |
| ↳↳ a. Locate the Kubeconfig File | [53](notes/53-tls-inside-the-cluster.md) |
| ↳↳ b. Inspect the CA Data | [53](notes/53-tls-inside-the-cluster.md) |
| CLIENT-SIDE: How the Scheduler Authenticates to the API Server | [53](notes/53-tls-inside-the-cluster.md) |
| ↳ Step 1: How Does the Scheduler Authenticate Itself? | [53](notes/53-tls-inside-the-cluster.md) |
| ↳ Step 2: How Does the API Server Verify the Scheduler? | [53](notes/53-tls-inside-the-cluster.md) |
| Conclusion: Focus on Config Files, Not Hardcoded Paths | [53](notes/53-tls-inside-the-cluster.md) |
| ↳ Summary Table | [53](notes/53-tls-inside-the-cluster.md) |
| Example 2: mTLS Between API Server (Client) and etcd (Server) | [53](notes/53-tls-inside-the-cluster.md) |
| SERVER-SIDE: etcd Presents Its Certificate to API Server | [53](notes/53-tls-inside-the-cluster.md) |
| ↳ Step 1: Know Who’s the Server | [53](notes/53-tls-inside-the-cluster.md) |
| ↳ Step 2: Where Is etcd's Certificate Defined? | [53](notes/53-tls-inside-the-cluster.md) |
| ↳ Step 3: How Does the API Server Verify etcd's Certificate? | [53](notes/53-tls-inside-the-cluster.md) |
| CLIENT-SIDE: API Server Authenticates to etcd | [53](notes/53-tls-inside-the-cluster.md) |
| ↳ Step 1: Where Does API Server Specify TLS Credentials? | [53](notes/53-tls-inside-the-cluster.md) |
| ↳ Step 2: How Does etcd Verify the API Server's Certificate? | [53](notes/53-tls-inside-the-cluster.md) |
| ↳ How Mutual TLS (mTLS) Works Here | [53](notes/53-tls-inside-the-cluster.md) |
| ↳ Takeaway: Focus on Flags, Not File Paths | [53](notes/53-tls-inside-the-cluster.md) |
| ↳ Summary Table | [53](notes/53-tls-inside-the-cluster.md) |
| Example 3: mTLS Between API Server (Client) and Kubelet (Server) | [53](notes/53-tls-inside-the-cluster.md) |
| Who is the Client and Who is the Server? | [53](notes/53-tls-inside-the-cluster.md) |
| SERVER-SIDE: How the Kubelet Presents Its Certificate | [53](notes/53-tls-inside-the-cluster.md) |
| ↳ Step 1: Locate the Kubelet Process | [53](notes/53-tls-inside-the-cluster.md) |
| ↳ Step 2: Inspect the Server Certificate | [53](notes/53-tls-inside-the-cluster.md) |
| ↳ How Did the Kubelet Get This Certificate? | [53](notes/53-tls-inside-the-cluster.md) |
| ↳ Step 3: How the API Server Verifies the Kubelet (or Doesn’t) | [53](notes/53-tls-inside-the-cluster.md) (rewritten paragraph corrected: C2) |
| CLIENT-SIDE: How the API Server Presents Its Own Certificate to the Kubelet | [53](notes/53-tls-inside-the-cluster.md) |
| ↳ How the Kubelet Verifies the API Server's Certificate | [53](notes/53-tls-inside-the-cluster.md) |
| Summary: mTLS Between API Server and Kubelet | [53](notes/53-tls-inside-the-cluster.md) |
| Recap: How mTLS Happens Here | [53](notes/53-tls-inside-the-cluster.md) |
| Tasks for You | [53](notes/53-tls-inside-the-cluster.md) |
| ↳↳ Task 1: Controller Manager (Client) → API Server (Server) | [53](notes/53-tls-inside-the-cluster.md) |
| ↳↳ Task 2: Kubelet (Client) → API Server (Server) | [53](notes/53-tls-inside-the-cluster.md) |
| ↳↳ Task 3: Kube-Proxy (Client) → API Server (Server) | [53](notes/53-tls-inside-the-cluster.md) |
| ↳ Granting Cluster Access to a New User (Seema) using Certificates and RBAC | [57](notes/57-rbac.md) [54](notes/54-authentication.md) |
| Conclusion | Not carried over as content: a summary of the course flow, which the learning path in [INDEX.md](INDEX.md) already gives |
| ↳ References | [50](notes/50-encryption-basics.md) [51](notes/51-certificates-and-cas.md) [52](notes/52-kubeconfig-deep-dive-and-mtls.md) [53](notes/53-tls-inside-the-cluster.md) [54](notes/54-authentication.md) (the same reference links as Days 30–34) |

| Image | Used in |
|---|---|
| `30a.png` | [50](notes/50-encryption-basics.md)  |
| `30b.png` | [50](notes/50-encryption-basics.md)  |
| `30c.png` | [50](notes/50-encryption-basics.md)  |
| `30d.png` | [50](notes/50-encryption-basics.md)  |
| `30e.png` | [50](notes/50-encryption-basics.md)  |
| `30f.png` | [50](notes/50-encryption-basics.md)  |
| `31-1.png` | [51](notes/51-certificates-and-cas.md)  |
| `31-10.png` | [52](notes/52-kubeconfig-deep-dive-and-mtls.md)  |
| `31-11.png` | Seema's kubectl ↔ API server TLS with the Kubernetes CA | [51](notes/51-certificates-and-cas.md) |
| `31-12.png` | [52](notes/52-kubeconfig-deep-dive-and-mtls.md)  |
| `31-2.png` | [51](notes/51-certificates-and-cas.md)  |
| `31-3.png` | [51](notes/51-certificates-and-cas.md)  |
| `31-4.png` | [51](notes/51-certificates-and-cas.md)  |
| `31-5.png` | [51](notes/51-certificates-and-cas.md)  |
| `31-6.png` | [51](notes/51-certificates-and-cas.md)  |
| `31-7.png` | [51](notes/51-certificates-and-cas.md)  |
| `31-8.png` | [51](notes/51-certificates-and-cas.md)  |
| `31-9.png` | [52](notes/52-kubeconfig-deep-dive-and-mtls.md)  |
| `33a.png` | [53](notes/53-tls-inside-the-cluster.md)  |
| `33b.png` | [53](notes/53-tls-inside-the-cluster.md)  |
| `33c.png` | [53](notes/53-tls-inside-the-cluster.md)  |
| `34a.png` | [54](notes/54-authentication.md)  |

## Day-by-day mapping

### Day 01 · Docker Fundamentals for Kubernetes | CKA Certification Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 1 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| How Were Development and Deployment Done Before Docker? Why Do We Need Docker? What Challenges Does It Solve? | [02](notes/02-docker-recap-containers-and-images.md)  |
| ↳ 1. Development | [02](notes/02-docker-recap-containers-and-images.md)  |
| ↳ 2. Testing | [02](notes/02-docker-recap-containers-and-images.md)  |
| ↳ 3. Deployment | [02](notes/02-docker-recap-containers-and-images.md)  |
| ↳ Challenges with Traditional Deployment Approaches: | [02](notes/02-docker-recap-containers-and-images.md)  |
| What Is Docker? | [02](notes/02-docker-recap-containers-and-images.md)  |
| ↳ Before Docker | [02](notes/02-docker-recap-containers-and-images.md)  |
| ↳ With Docker | [02](notes/02-docker-recap-containers-and-images.md)  |
| ↳ Understanding Docker Containers Through Shipping Containers | [02](notes/02-docker-recap-containers-and-images.md)  |
| ↳ Shipping Containers vs. Docker Containers | [02](notes/02-docker-recap-containers-and-images.md)  |
| Virtual Machines (VMs) vs Docker Architecture | [02](notes/02-docker-recap-containers-and-images.md)  |
| High-Level Workflow with Docker | [02](notes/02-docker-recap-containers-and-images.md)  |
| Low-Level Workflow with Docker | [02](notes/02-docker-recap-containers-and-images.md)  |
| ↳ Key Concepts: | [02](notes/02-docker-recap-containers-and-images.md)  |
| ↳ Docker Engine Components | [02](notes/02-docker-recap-containers-and-images.md)  |
| ↳ Using the Car Analogy | [02](notes/02-docker-recap-containers-and-images.md)  |
| ↳ Key Steps in the Workflow: | [02](notes/02-docker-recap-containers-and-images.md)  |
| ↳ Conclusion | [02](notes/02-docker-recap-containers-and-images.md)  |

| Image | Used in |
|---|---|
| `1a.png` | [01](notes/01-why-kubernetes.md)  |
| `1b.png` | [01](notes/01-why-kubernetes.md)  |
| `1c.png` | [02](notes/02-docker-recap-containers-and-images.md)  |
| `1d.png` | [02](notes/02-docker-recap-containers-and-images.md)  |
| `1e.png` | [02](notes/02-docker-recap-containers-and-images.md)  |

### Day 02 · Write Your First Dockerfile, and Push to Docker Hub | CKA Certification Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 2 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Understanding docker pull docker.io/library/ubuntu:latest | [02](notes/02-docker-recap-containers-and-images.md)  |
| ↳ Pulling Images from Other Registries | [02](notes/02-docker-recap-containers-and-images.md)  |
| ↳ Key Takeaways: | [02](notes/02-docker-recap-containers-and-images.md)  |
| ↳ Important Docker Commands | [02](notes/02-docker-recap-containers-and-images.md)  |
| ↳ Final Notes | [02](notes/02-docker-recap-containers-and-images.md)  |

| Image | Used in |
|---|---|
| `2a.png` | [02](notes/02-docker-recap-containers-and-images.md)  |

| File | Used in |
|---|---|
| `Dockerfile` | [02](notes/02-docker-recap-containers-and-images.md) [03](notes/03-docker-recap-dockerfiles.md)  |

### Day 03 · Docker Flags, Deep Dive into Dockerfile, and Exposing Containers | CKA Certification Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 3 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ↳ Important Docker Flags | [03](notes/03-docker-recap-dockerfiles.md)  |
| ↳ Dockerfile Instructions and Their Purposes | [03](notes/03-docker-recap-dockerfiles.md)  |
| ↳ Example Dockerfile | [03](notes/03-docker-recap-dockerfiles.md)  |
| ↳ Shell Form vs Exec Form in CMD | [03](notes/03-docker-recap-dockerfiles.md)  |
| ↳ Conclusion | [03](notes/03-docker-recap-dockerfiles.md)  |

| Image | Used in |
|---|---|
| `3a.png` | [03](notes/03-docker-recap-dockerfiles.md)  |

| File | Used in |
|---|---|
| `app.py` | [03](notes/03-docker-recap-dockerfiles.md) [36](notes/36-docker-storage-recap.md)  |
| `Dockerfile` | [03](notes/03-docker-recap-dockerfiles.md) [36](notes/36-docker-storage-recap.md)  |

### Day 04 · Docker Flags, Deep Dive into Dockerfile, and Exposing Containers | CKA Certification Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 4 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Specifying a Custom Dockerfile Name and Understanding the Build Command in Docker | [03](notes/03-docker-recap-dockerfiles.md)  |
| Why Use a Custom Name for Dockerfile? | [03](notes/03-docker-recap-dockerfiles.md)  |
| Building an Image with a Custom Dockerfile Name | [03](notes/03-docker-recap-dockerfiles.md)  |
| ↳ Command Syntax | [03](notes/03-docker-recap-dockerfiles.md)  |
| ↳ Explanation of Flags | [03](notes/03-docker-recap-dockerfiles.md)  |
| Understanding the Build Context | [03](notes/03-docker-recap-dockerfiles.md)  |
| ↳ Key Points About the Build Context | [03](notes/03-docker-recap-dockerfiles.md)  |
| ↳ Example: Building with a Custom Build Context | [03](notes/03-docker-recap-dockerfiles.md)  |
| CMD vs ENTRYPOINT | [03](notes/03-docker-recap-dockerfiles.md)  |
| Common Docker Commands | [02](notes/02-docker-recap-containers-and-images.md)  |
| ↳ Container Management | [02](notes/02-docker-recap-containers-and-images.md)  |
| ↳ Image Management | [02](notes/02-docker-recap-containers-and-images.md)  |
| ↳ Conclusion | [02](notes/02-docker-recap-containers-and-images.md)  |

| File | Used in |
|---|---|
| `cmd` | [03](notes/03-docker-recap-dockerfiles.md)  |
| `ec` | [02](notes/02-docker-recap-containers-and-images.md) [03](notes/03-docker-recap-dockerfiles.md)  |
| `entrypoint` | [03](notes/03-docker-recap-dockerfiles.md)  |
| `entrypoint-practice` | [03](notes/03-docker-recap-dockerfiles.md)  |

### Day 05 · Docker Multi-Stage Builds & Image Optimization | CKA Certification Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 5 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Compiled vs. Interpreted Languages | [03](notes/03-docker-recap-dockerfiles.md)  |
| ↳ Compiled Languages | [03](notes/03-docker-recap-dockerfiles.md)  |
| ↳ Interpreted Languages | [03](notes/03-docker-recap-dockerfiles.md)  |
| ↳ Build Stage vs. Runtime Stage | [03](notes/03-docker-recap-dockerfiles.md)  |
| ↳ Pizza Analogy | [03](notes/03-docker-recap-dockerfiles.md)  |
| ↳ Benefits of Separating Build and Runtime Stages | [03](notes/03-docker-recap-dockerfiles.md)  |
| Dockerfile Example: Multi-Stage Build | [03](notes/03-docker-recap-dockerfiles.md)  |
| ↳ Docker Multi-Stage Builds | [03](notes/03-docker-recap-dockerfiles.md)  |
| ↳ Instructions that Create and Do Not Create Layers | [03](notes/03-docker-recap-dockerfiles.md)  |
| ↳ Best Practices | [03](notes/03-docker-recap-dockerfiles.md)  |
| ↳ Conclusion | [03](notes/03-docker-recap-dockerfiles.md)  |

| File | Used in |
|---|---|
| `app.py` | [03](notes/03-docker-recap-dockerfiles.md)  |
| `Dockerfile` | [03](notes/03-docker-recap-dockerfiles.md)  |
| `Dockerfile.multi` | [03](notes/03-docker-recap-dockerfiles.md)  |

### Day 06 · Hello Kubernetes! Why Kubernetes? What is Kubernetes? | CKA Certification Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 6 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Challenges with Docker | [01](notes/01-why-kubernetes.md)  |
| Hello Kubernetes!! | [01](notes/01-why-kubernetes.md)  |
| ↳ Why Kubernetes? | [01](notes/01-why-kubernetes.md)  |
| Kubernetes vs Docker Swarm vs Docker Compose | [01](notes/01-why-kubernetes.md)  |
| ↳ Key pointers covering the "Characteristics" we discussed? | [01](notes/01-why-kubernetes.md)  |

| Image | Used in |
|---|---|
| `6a.png` | [01](notes/01-why-kubernetes.md)  |
| `6b.png` | [01](notes/01-why-kubernetes.md)  |

### Day 07 · Kubernetes Architecture & Deployment Creation Workflow | CKA Certification Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 7 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| What is Pod? | [18](notes/18-pods.md)  |
| What are Network namespaces? | [04](notes/04-linux-under-kubernetes.md)  |
| What is a Deployment? | [06](notes/06-the-cluster-at-a-glance.md)  |
| ↳ Control Plane | [06](notes/06-the-cluster-at-a-glance.md)  |
| ↳ Worker Nodes | [06](notes/06-the-cluster-at-a-glance.md)  |
| Control Plane Components | [07](notes/07-control-plane-components.md)  |
| Data Plane Components | [07](notes/07-control-plane-components.md)  |
| Control Plane vs Data Plane | [07](notes/07-control-plane-components.md)  |
| Kubernetes: Python Frontend, Redis Service, kube-proxy, and CNI Interaction | [32](notes/32-inside-a-service-kube-proxy.md) [29](notes/29-network-model-and-cni.md)  |
| Example Flow: Communication Between Pods | [32](notes/32-inside-a-service-kube-proxy.md) [29](notes/29-network-model-and-cni.md)  |
| ↳ Step-by-Step Flow: | [32](notes/32-inside-a-service-kube-proxy.md) [29](notes/29-network-model-and-cni.md)  |
| 1. User Initiates Deployment Creation | [10](notes/10-what-happens-after-kubectl-apply.md)  |
| 2. API Server Actions | [10](notes/10-what-happens-after-kubectl-apply.md)  |
| 3. Deployment Controller Workflow | [10](notes/10-what-happens-after-kubectl-apply.md)  |
| ↳ How It Works | [10](notes/10-what-happens-after-kubectl-apply.md)  |
| ↳ What Happens When Desired State Differs? | [10](notes/10-what-happens-after-kubectl-apply.md)  |
| 4. Scheduler's Role | [10](notes/10-what-happens-after-kubectl-apply.md)  |
| ↳ How It Works | [10](notes/10-what-happens-after-kubectl-apply.md)  |
| 5. API Server Updates the Pod Status | [10](notes/10-what-happens-after-kubectl-apply.md)  |
| 6. Kubelet's Role | [10](notes/10-what-happens-after-kubectl-apply.md)  |
| ↳ How It Works | [10](notes/10-what-happens-after-kubectl-apply.md)  |
| ↳ Kubelet Actions | [10](notes/10-what-happens-after-kubectl-apply.md)  |
| 7. Pod Running | [10](notes/10-what-happens-after-kubectl-apply.md)  |
| ↳ Additional Resources | [10](notes/10-what-happens-after-kubectl-apply.md)  |

| Image | Used in |
|---|---|
| `7a.png` | [18](notes/18-pods.md)  |
| `7b.png` | [06](notes/06-the-cluster-at-a-glance.md)  |
| `7c.png` | [06](notes/06-the-cluster-at-a-glance.md)  |
| `7d.png` | [32](notes/32-inside-a-service-kube-proxy.md)  |
| `7e.png` | [10](notes/10-what-happens-after-kubectl-apply.md)  |
| `7f.png` | [06](notes/06-the-cluster-at-a-glance.md)  |

### Day 08 · Setting Up Kind Cluster Locally & Kubernetes Context | CKA Certification Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 8 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| First Scenario: *my-first-cluster* | [11](notes/11-lab-setup-kind-and-minikube.md)  |
| Second Scenario: *my-second-cluster* | [11](notes/11-lab-setup-kind-and-minikube.md)  |
| Important Note About Naming in Kind | [11](notes/11-lab-setup-kind-and-minikube.md)  |
| Scenario: | [12](notes/12-kubectl-kubeconfig-and-contexts.md)  |
| How Kubernetes Contexts Work: | [12](notes/12-kubectl-kubeconfig-and-contexts.md)  |
| ↳ 1. Configuring Contexts: | [12](notes/12-kubectl-kubeconfig-and-contexts.md)  |
| ↳ 2. Switching Between Contexts: | [12](notes/12-kubectl-kubeconfig-and-contexts.md)  |
| Example kubeconfig File: | [12](notes/12-kubectl-kubeconfig-and-contexts.md)  |
| Leveraging Contexts Properly: | [12](notes/12-kubectl-kubeconfig-and-contexts.md)  |
| Kubernetes Origin | [01](notes/01-why-kubernetes.md)  |
| Additional Resources: | [11](notes/11-lab-setup-kind-and-minikube.md)  |

| Image | Used in |
|---|---|
| `8a.png` | [12](notes/12-kubectl-kubeconfig-and-contexts.md)  |
| `8b.png` | [01](notes/01-why-kubernetes.md)  |

| File | Used in |
|---|---|
| `kind-cluster.yaml` | [11](notes/11-lab-setup-kind-and-minikube.md)  |

### Day 09 · YAML Tutorial for Kubernetes | Imperative vs Declarative | CKA Certification Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 9 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Imperative vs. Declarative | [15](notes/15-imperative-vs-declarative.md)  |
| ↳ Imperative vs. Declarative - Table | [15](notes/15-imperative-vs-declarative.md)  |
| Introduction to YAML | [13](notes/13-yaml-and-manifest-anatomy.md)  |
| ↳ YAML vs JSON vs XML | [13](notes/13-yaml-and-manifest-anatomy.md)  |
| ↳ Basic YAML Data Structures | [13](notes/13-yaml-and-manifest-anatomy.md)  |
| ↳ Kubernetes Pod Manifest Example | [13](notes/13-yaml-and-manifest-anatomy.md)  |
| ↳ Different Names for Kubernetes Configuration Files | [13](notes/13-yaml-and-manifest-anatomy.md)  |
| ↳ Basic Pod-Related Commands | [18](notes/18-pods.md)  |

| Image | Used in |
|---|---|
| `9a.png` | [13](notes/13-yaml-and-manifest-anatomy.md)  |
| `9b.png` | [15](notes/15-imperative-vs-declarative.md)  |

| File | Used in |
|---|---|
| `pod.yaml` | [13](notes/13-yaml-and-manifest-anatomy.md) [18](notes/18-pods.md)  |
| `practice.yaml` | [13](notes/13-yaml-and-manifest-anatomy.md)  |

### Day 10 · Replication Controller, ReplicaSets, and Deployment | CKA Certification Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 10 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| 1. Checking Kubernetes Object Details | [24](notes/24-replicationcontroller-and-replicaset.md) [17](notes/17-labels-selectors-and-annotations.md)  |
| 2. ReplicationController (rc) | [24](notes/24-replicationcontroller-and-replicaset.md)  |
| ↳ What is a ReplicationController? | [24](notes/24-replicationcontroller-and-replicaset.md)  |
| ↳ Labels and Selectors | [17](notes/17-labels-selectors-and-annotations.md)  |
| ↳ rc Manifest Example | [24](notes/24-replicationcontroller-and-replicaset.md)  |
| ↳ How rc Works | [24](notes/24-replicationcontroller-and-replicaset.md)  |
| ↳ Imperative Way to Create an rc (deprecated) | [24](notes/24-replicationcontroller-and-replicaset.md)  |
| 3. ReplicaSet (rs) | [24](notes/24-replicationcontroller-and-replicaset.md)  |
| ↳ What is a ReplicaSet? | [24](notes/24-replicationcontroller-and-replicaset.md)  |
| ↳ rs Manifest Example | [24](notes/24-replicationcontroller-and-replicaset.md)  |
| ↳ Equality-Based vs Set-Based Selectors | [17](notes/17-labels-selectors-and-annotations.md)  |
| ↳ Explanation: | [17](notes/17-labels-selectors-and-annotations.md)  |
| 4. Deployments | [25](notes/25-deployments.md)  |
| ↳ What is a Deployment? | [25](notes/25-deployments.md)  |
| ↳ Deployment Manifest Example | [25](notes/25-deployments.md)  |
| ↳ How Deployments Build on rs | [25](notes/25-deployments.md)  |
| ↳ Explaining Rolling Updates and Rollbacks in Deployments with Annotations | [25](notes/25-deployments.md)  |
| ↳ Key Pointers: | [25](notes/25-deployments.md)  |
| ↳ Important Kubernetes Deployment Commands | [25](notes/25-deployments.md)  |
| 5. Key Differences Between rc, rs, and Deployments | [25](notes/25-deployments.md)  |
| 6. References | [24](notes/24-replicationcontroller-and-replicaset.md)  |

| Image | Used in |
|---|---|
| `10a.png` | [24](notes/24-replicationcontroller-and-replicaset.md)  |
| `10b.png` | [24](notes/24-replicationcontroller-and-replicaset.md)  |
| `10c.png` | [17](notes/17-labels-selectors-and-annotations.md)  |
| `10d.png` | [17](notes/17-labels-selectors-and-annotations.md)  |
| `10e.png` | [17](notes/17-labels-selectors-and-annotations.md) [24](notes/24-replicationcontroller-and-replicaset.md)  |
| `10f.png` | [25](notes/25-deployments.md)  |

| File | Used in |
|---|---|
| `deploy.yaml` | [25](notes/25-deployments.md)  |
| `rc.yaml` | [24](notes/24-replicationcontroller-and-replicaset.md)  |
| `rs.yaml` | [24](notes/24-replicationcontroller-and-replicaset.md)  |
| `rs-set-based.yaml` | [24](notes/24-replicationcontroller-and-replicaset.md)  |
| `set-based.yaml` | [17](notes/17-labels-selectors-and-annotations.md) [24](notes/24-replicationcontroller-and-replicaset.md)  |

### Day 11 · Microservices & 3-Tier Architecture | Software Design for Kubernetes | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 11 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ↳ Monolithic Architecture | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| 1. Three-Tier Architecture Overview | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| 2. Understanding Each Tier in Detail | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ Frontend (Web Tier) | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ Middleware | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ Backend (Application Tier) | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ Database (Data Tier) | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ 3-Tier Deployment Models | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ A. Traditional Single-Server Deployment | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ B. Moving to a Three-Server Model (Pre-Virtualization Era) | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ C. How Virtualization Enabled Widespread 3-Tier Adoption | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ D. The Shift to Cloud and Kubernetes | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| Why Do We Need Microservices? | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ Example: Monolithic E-Commerce Application | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ Microservices: A Better Alternative | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ Benefits of Microservices | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ Mapping Microservices to a Three-Tier Architecture | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ Database (Data Tier) – Where Should Databases Be Hosted? | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ Why Are Databases Usually Not Hosted in Kubernetes? | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ Where Are Databases Hosted in Production? | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ When Would You Run a Database in Kubernetes? | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ Example: How an E-Commerce Application Works in Kubernetes | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ Scaling Microservices in Kubernetes | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |

| Image | Used in |
|---|---|
| `11a.png` | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| `11b.png` | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| `11c.png` | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |

### Day 12 · Kubernetes Services IN-DEPTH | ClusterIP, NodePort, LoadBalancer, ExternalName | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 11 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| The Problem: How Do Pods Communicate? | [30](notes/30-services-clusterip.md)  |
| ↳ Challenge 1: Pod IPs Keep Changing | [30](notes/30-services-clusterip.md)  |
| ↳ Challenge 2: Pods Need to Expose Their Services | [30](notes/30-services-clusterip.md)  |
| The Solution: Kubernetes Services | [30](notes/30-services-clusterip.md)  |
| Example: How Services Enable Communication in Kubernetes | [30](notes/30-services-clusterip.md)  |
| ↳ Step-1: User Requests the Frontend Application | [30](notes/30-services-clusterip.md)  |
| ↳ Step-2: Frontend Communicates with Backend | [30](notes/30-services-clusterip.md)  |
| Visualizing Service-Based Communication | [30](notes/30-services-clusterip.md)  |
| ↳ Without Services (Doesn’t Work) | [30](notes/30-services-clusterip.md)  |
| ↳ With Services (Works) | [30](notes/30-services-clusterip.md)  |
| Key Takeways: | [30](notes/30-services-clusterip.md)  |
| What is a ClusterIP Service? | [30](notes/30-services-clusterip.md)  |
| ↳ Analogy: | [30](notes/30-services-clusterip.md)  |
| ↳ Relating to Kubernetes: | [30](notes/30-services-clusterip.md)  |
| ↳ Key Characteristics of ClusterIP Service | [30](notes/30-services-clusterip.md)  |
| Example: Demonstrating ClusterIP with Frontend (NGINX) and Backend (http-echo) | [30](notes/30-services-clusterip.md)  |
| ↳ Architecture Overview | [30](notes/30-services-clusterip.md)  |
| ↳ How the Frontend Communicates with the Backend | [30](notes/30-services-clusterip.md)  |
| Deploying the Frontend and Backend | [30](notes/30-services-clusterip.md)  |
| ↳ Frontend Deployment (NGINX) | [30](notes/30-services-clusterip.md)  |
| ↳ Backend Deployment (http-echo) | [30](notes/30-services-clusterip.md)  |
| ↳ Backend Service (ClusterIP) | [30](notes/30-services-clusterip.md)  |
| Key Takeaways: | [30](notes/30-services-clusterip.md)  |
| Step 1: Check Existing KIND Clusters | [11](notes/11-lab-setup-kind-and-minikube.md)  |
| Step 2: Delete the Existing Cluster (If Any) | [11](notes/11-lab-setup-kind-and-minikube.md)  |
| Step 3: Create a New KIND Cluster | [11](notes/11-lab-setup-kind-and-minikube.md)  |
| ↳ 3.1 Use the Below Configuration File (kind-cluster.yaml) | [11](notes/11-lab-setup-kind-and-minikube.md)  |
| ↳ Create the Cluster Using This Configuration | [11](notes/11-lab-setup-kind-and-minikube.md)  |
| Understanding extraPortMappings Configuration | [11](notes/11-lab-setup-kind-and-minikube.md)  |
| ↳ Breakdown of Each Field | [11](notes/11-lab-setup-kind-and-minikube.md)  |
| ↳ Example Use Case | [11](notes/11-lab-setup-kind-and-minikube.md)  |
| What is a NodePort Service? | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| ↳ Analogy: | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| ↳ Relating to Kubernetes: | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| ↳ Key Characteristics of NodePort Service | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| ↳ Exposing a Frontend Application Using a NodePort Service | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| ↳ Step 1: Frontend Deployment (frontend.yaml) | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| ↳ Step 2: Frontend NodePort Service (frontend-service.yaml) | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| How a NodePort Service Handles Requests (Step-by-Step Flow) | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| ↳ Request Flow: | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| ↳ What is a LoadBalancer Service? | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| ↳ Analogy: | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| ↳ Relating to Kubernetes: | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| LoadBalancer Service Manifest Example | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| ↳ How This Works Behind the Scenes: | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| ↳ What is an ExternalName Service? | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| ↳ Analogy: | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| ↳ Relating to Kubernetes: | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| ExternalName Service Manifest Example | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| ↳ Example Scenario: | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| ↳ Why is This Useful? | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| When to Use an ExternalName Service: | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| ↳ ClusterIP (Internal-Only Communication) | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| ↳ NodePort (External Access with Manual IP Management) | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| ↳ LoadBalancer (Single Public IP with Automatic Failover) | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| ↳ ExternalName (Internal Service to External Service Mapping) | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| Key Takeaways: | [30](notes/30-services-clusterip.md)  |
| References | [30](notes/30-services-clusterip.md)  |

| Image | Used in |
|---|---|
| `12a.png` | [30](notes/30-services-clusterip.md)  |
| `12b.png` | [30](notes/30-services-clusterip.md)  |
| `12c.png` | [30](notes/30-services-clusterip.md)  |
| `12d.png` | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| `12e.png` | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| `12f.png` | [11](notes/11-lab-setup-kind-and-minikube.md)  |
| `12g.png` | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| `12h.png` | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| `12j.png` | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| `12k.png` | [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |

| File | Used in |
|---|---|
| `backend.yaml` | [30](notes/30-services-clusterip.md)  |
| `frontend.yaml` | [30](notes/30-services-clusterip.md) [31](notes/31-services-nodeport-loadbalancer-externalname-headless.md)  |
| `kind-cluster.yaml` | [11](notes/11-lab-setup-kind-and-minikube.md)  |

### Day 13 · Imperative commands for Deployments, Services and Troubleshooting | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 13 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Why Are Imperative Commands Important? | [15](notes/15-imperative-vs-declarative.md)  |
| ↳ Important Flags to Know: | [15](notes/15-imperative-vs-declarative.md)  |
| Imperative Commands for Backend Deployment and Service | [30](notes/30-services-clusterip.md)  |
| ↳ Step 1: Create Backend Deployment | [30](notes/30-services-clusterip.md)  |
| ↳ Step 2: Create Backend Service | [30](notes/30-services-clusterip.md)  |
| ↳ Manual Steps: | [30](notes/30-services-clusterip.md)  |
| Imperative Commands for Frontend Deployment and Service | [30](notes/30-services-clusterip.md)  |
| ↳ Step 1: Create Frontend Deployment | [30](notes/30-services-clusterip.md)  |
| ↳ Step 2: Create Frontend Service | [30](notes/30-services-clusterip.md)  |
| ↳ Manual Steps: | [30](notes/30-services-clusterip.md)  |
| Labels: What You Need to Know | [17](notes/17-labels-selectors-and-annotations.md)  |
| ↳ Verify Labels: | [17](notes/17-labels-selectors-and-annotations.md)  |
| Creating Test Pods Imperatively | [15](notes/15-imperative-vs-declarative.md)  |
| ↳ Imperative Command to Create a Pod: | [15](notes/15-imperative-vs-declarative.md)  |
| ↳ Create a Temporary Pod for Troubleshooting: | [15](notes/15-imperative-vs-declarative.md)  |
| Key Takeaways: | [30](notes/30-services-clusterip.md)  |

| File | Used in |
|---|---|
| `backend-deploy.yaml` | [15](notes/15-imperative-vs-declarative.md) [30](notes/30-services-clusterip.md)  |
| `frontend-deploy.yaml` | [15](notes/15-imperative-vs-declarative.md) [30](notes/30-services-clusterip.md)  |

### Day 14 · Kubernetes Namespaces Explained | Isolation & Resource Management | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 14 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| What Are Namespaces in Kubernetes? | [26](notes/26-namespaces.md)  |
| Analogy to Understand Namespaces | [26](notes/26-namespaces.md)  |
| ↳ Relating to Kubernetes: | [26](notes/26-namespaces.md)  |
| Why Use Namespaces? | [26](notes/26-namespaces.md)  |
| Default Namespaces in Kubernetes | [26](notes/26-namespaces.md)  |
| Working with Namespaces | [26](notes/26-namespaces.md)  |
| ↳ Creating Namespaces | [26](notes/26-namespaces.md)  |
| ↳ Viewing and Deleting Namespaces | [26](notes/26-namespaces.md)  |
| ↳ Using Namespace Flags | [26](notes/26-namespaces.md)  |
| Deploying Frontend and Backend in a Namespace | [26](notes/26-namespaces.md)  |
| ↳ 1. Applying Frontend and Backend YAMLs | [26](notes/26-namespaces.md)  |
| ↳ 2. Verify the Deployments in app1-ns | [26](notes/26-namespaces.md)  |
| Testing Namespace Isolation | [26](notes/26-namespaces.md)  |
| ↳ 1. Test Pod in default Namespace | [26](notes/26-namespaces.md)  |
| ↳ ✅ Format: | [26](notes/26-namespaces.md)  |
| ↳ 💡 Coming Up: | [26](notes/26-namespaces.md)  |
| ↳ 2. Test Pod in app1-ns Namespace | [26](notes/26-namespaces.md)  |
| Setting a Default Namespace in Kubernetes Context | [26](notes/26-namespaces.md)  |
| ↳ Why? | [26](notes/26-namespaces.md)  |
| Best Practices for Using Namespaces | [26](notes/26-namespaces.md)  |
| Summary | [26](notes/26-namespaces.md)  |
| References | [26](notes/26-namespaces.md)  |

| File | Used in |
|---|---|
| `app1-ns.yaml` | [26](notes/26-namespaces.md)  |
| `backend-deploy.yaml` | [26](notes/26-namespaces.md)  |
| `frontend-deploy.yaml` | [26](notes/26-namespaces.md)  |

### Day 15 · Manual Scheduling & Static Pods | MUST KNOW Concepts | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 15 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Prerequisite: Revisiting Kubernetes Deployment Workflow | [10](notes/10-what-happens-after-kubectl-apply.md)  |
| Understanding the Kubernetes Scheduler | [42](notes/42-scheduler-manual-scheduling-static-pods.md)  |
| Manual Scheduling | [42](notes/42-scheduler-manual-scheduling-static-pods.md)  |
| ↳ Why is Manual Scheduling Required? | [42](notes/42-scheduler-manual-scheduling-static-pods.md)  |
| ↳ What is Manual Scheduling? | [42](notes/42-scheduler-manual-scheduling-static-pods.md)  |
| ↳ How is Manual Scheduling Useful? | [42](notes/42-scheduler-manual-scheduling-static-pods.md)  |
| ↳ Demonstration: Assigning a Pod to a Node | [42](notes/42-scheduler-manual-scheduling-static-pods.md)  |
| ↳ Running a Pod on the Control Plane | [42](notes/42-scheduler-manual-scheduling-static-pods.md)  |
| ↳ How Can Control Plane Run Workloads? | [42](notes/42-scheduler-manual-scheduling-static-pods.md)  |
| Static Pods | [42](notes/42-scheduler-manual-scheduling-static-pods.md)  |
| ↳ Why Do We Need Static Pods? | [42](notes/42-scheduler-manual-scheduling-static-pods.md)  |
| ↳ What Are Static Pods? | [42](notes/42-scheduler-manual-scheduling-static-pods.md)  |
| ↳ Mirror Pods in Kubernetes | [42](notes/42-scheduler-manual-scheduling-static-pods.md)  |
| ↳ Demonstration: Creating a Static Pod | [42](notes/42-scheduler-manual-scheduling-static-pods.md)  |
| ↳ Why Does kubectl delete Not Permanently Remove Static Pods? | [42](notes/42-scheduler-manual-scheduling-static-pods.md)  |
| ↳ Accessing Static Pods in Production vs. KIND | [42](notes/42-scheduler-manual-scheduling-static-pods.md)  |
| Key Differences Between Manual Scheduling & Static Pods | [42](notes/42-scheduler-manual-scheduling-static-pods.md)  |
| Summary | [42](notes/42-scheduler-manual-scheduling-static-pods.md)  |
| References | [42](notes/42-scheduler-manual-scheduling-static-pods.md)  |

| Image | Used in |
|---|---|
| `15a.png` | [42](notes/42-scheduler-manual-scheduling-static-pods.md)  |

| File | Used in |
|---|---|
| `manual-pod.yaml` | [42](notes/42-scheduler-manual-scheduling-static-pods.md)  |

### Day 16 · Mastering Kubernetes Taints & Tolerations | Essential Scheduling Control | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 16 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Introduction: Why Taints and Tolerations? | [43](notes/43-taints-and-tolerations.md)  |
| ↳ When Do We Need Taints & Tolerations? | [43](notes/43-taints-and-tolerations.md)  |
| ↳ Important: | [43](notes/43-taints-and-tolerations.md)  |
| Understanding Taints | [43](notes/43-taints-and-tolerations.md)  |
| ↳ How to Apply a Taint? | [43](notes/43-taints-and-tolerations.md)  |
| ↳ Effects of a Taint | [43](notes/43-taints-and-tolerations.md)  |
| Understanding Tolerations | [43](notes/43-taints-and-tolerations.md)  |
| ↳ How to Apply a Toleration? | [43](notes/43-taints-and-tolerations.md)  |
| ↳ Toleration Operators | [43](notes/43-taints-and-tolerations.md)  |
| How Taints and Tolerations Work Together | [43](notes/43-taints-and-tolerations.md)  |
| Demonstration | [43](notes/43-taints-and-tolerations.md)  |
| ↳ Cluster Setup for Demonstration | [43](notes/43-taints-and-tolerations.md)  |
| ↳ Applying Taints | [43](notes/43-taints-and-tolerations.md)  |
| ↳ Verifying Taints | [43](notes/43-taints-and-tolerations.md)  |
| ↳ Checking Pod Behavior Without Tolerations | [43](notes/43-taints-and-tolerations.md)  |
| ↳ Why the Pod Is Not Scheduled on the Control Plane Node? | [43](notes/43-taints-and-tolerations.md)  |
| ↳ Why Were We Able to Manually Schedule a Pod on the Control Plane in the Previous Lecture? | [43](notes/43-taints-and-tolerations.md)  |
| ↳ Scheduling Pods with Tolerations | [43](notes/43-taints-and-tolerations.md)  |
| ↳ Using the Exists Operator | [43](notes/43-taints-and-tolerations.md)  |
| ↳ Applying Multiple Taints and Tolerations | [43](notes/43-taints-and-tolerations.md)  |
| ↳ Deleting Taints | [43](notes/43-taints-and-tolerations.md)  |
| Scenario: A Cluster with Colored Nodes | [43](notes/43-taints-and-tolerations.md)  |
| Effect of Taints on Existing Pods | [43](notes/43-taints-and-tolerations.md)  |
| Pod Placement Behavior for New Pods | [43](notes/43-taints-and-tolerations.md)  |
| ↳ 1️⃣ Yellow Pod | [43](notes/43-taints-and-tolerations.md)  |
| ↳ 2️⃣ Normal Pod (No Toleration) | [43](notes/43-taints-and-tolerations.md)  |
| ↳ 3️⃣ Green Pod | [43](notes/43-taints-and-tolerations.md)  |
| ↳ 4️⃣ Blue Pod | [43](notes/43-taints-and-tolerations.md)  |
| Why Should Pods with Tolerations Prefer Tainted Nodes? | [43](notes/43-taints-and-tolerations.md)  |
| ↳ Real-World Use Cases for This Segregation | [43](notes/43-taints-and-tolerations.md)  |
| Key Takeaways | [43](notes/43-taints-and-tolerations.md)  |
| Reference | [43](notes/43-taints-and-tolerations.md)  |

| Image | Used in |
|---|---|
| `16a.png` | [43](notes/43-taints-and-tolerations.md)  |

| File | Used in |
|---|---|
| `deploy.yaml` | [43](notes/43-taints-and-tolerations.md)  |
| `deploy2.yaml` | [43](notes/43-taints-and-tolerations.md)  |

### Day 17 · Mastering Node Selector & Node Affinity Rules in Kubernetes | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 17 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Introduction | [44](notes/44-nodeselector-and-node-affinity.md)  |
| Node Placement Strategies | [44](notes/44-nodeselector-and-node-affinity.md)  |
| ↳ How It Works | [44](notes/44-nodeselector-and-node-affinity.md)  |
| Limitations of Node Selector | [44](notes/44-nodeselector-and-node-affinity.md)  |
| ↳ Cluster Setup for Demonstration | [44](notes/44-nodeselector-and-node-affinity.md)  |
| Demonstration: Node Selector | [44](notes/44-nodeselector-and-node-affinity.md)  |
| ↳ Step 1: Check Existing Labels on Nodes | [44](notes/44-nodeselector-and-node-affinity.md)  |
| ↳ Step 2: Label a Node | [44](notes/44-nodeselector-and-node-affinity.md)  |
| ↳ Step 3: Deploy a Pod with nodeSelector | [44](notes/44-nodeselector-and-node-affinity.md)  |
| ↳ Step 4: Add Multiple Labels to a Node | [44](notes/44-nodeselector-and-node-affinity.md)  |
| ↳ Step 5: Removing Labels | [44](notes/44-nodeselector-and-node-affinity.md)  |
| Key Takeaways for nodeSelector | [44](notes/44-nodeselector-and-node-affinity.md)  |
| Node Affinity in Kubernetes | [44](notes/44-nodeselector-and-node-affinity.md)  |
| Types of Node Affinity | [44](notes/44-nodeselector-and-node-affinity.md)  |
| Demo: Node Affinity | [44](notes/44-nodeselector-and-node-affinity.md)  |
| ↳ Step 1: Label a Node | [44](notes/44-nodeselector-and-node-affinity.md)  |
| ↳ Step 2: Deploy a Pod with Required Node Affinity | [44](notes/44-nodeselector-and-node-affinity.md)  |
| ↳ Expected Behavior | [44](notes/44-nodeselector-and-node-affinity.md)  |
| OR Condition (Multiple Label Matches) | [44](notes/44-nodeselector-and-node-affinity.md)  |
| ↳ Step 1: Label a Second Node | [44](notes/44-nodeselector-and-node-affinity.md)  |
| ↳ Step 2: Update Node Affinity to Allow Either SSD or HDD | [44](notes/44-nodeselector-and-node-affinity.md)  |
| ↳ Expected Behavior | [44](notes/44-nodeselector-and-node-affinity.md)  |
| AND Condition (Multiple Label Requirements) | [44](notes/44-nodeselector-and-node-affinity.md)  |
| ↳ Step 1: Label a Node | [44](notes/44-nodeselector-and-node-affinity.md)  |
| ↳ Step 2: Apply AND Condition | [44](notes/44-nodeselector-and-node-affinity.md)  |
| ↳ Expected Behavior | [44](notes/44-nodeselector-and-node-affinity.md)  |
| Node Affinity Operators | [44](notes/44-nodeselector-and-node-affinity.md)  |
| ↳ Example: Using All Operators | [44](notes/44-nodeselector-and-node-affinity.md)  |
| ↳ Explanation | [44](notes/44-nodeselector-and-node-affinity.md)  |
| Node Anti-Affinity | [44](notes/44-nodeselector-and-node-affinity.md)  |
| Summary | [44](notes/44-nodeselector-and-node-affinity.md)  |
| Understanding preferredDuringSchedulingIgnoredDuringExecution | [44](notes/44-nodeselector-and-node-affinity.md)  |
| ↳ How Node Affinity Weight Works | [44](notes/44-nodeselector-and-node-affinity.md)  |
| Demonstration: Using Preferred Node Affinity | [44](notes/44-nodeselector-and-node-affinity.md)  |
| ↳ Step 1: Apply Labels to Nodes | [44](notes/44-nodeselector-and-node-affinity.md)  |
| ↳ Step 2: Define Node Affinity with Preferences | [44](notes/44-nodeselector-and-node-affinity.md)  |
| ↳ Explanation | [44](notes/44-nodeselector-and-node-affinity.md)  |
| ↳ Step 3: Deploy and Observe Pod Placement | [44](notes/44-nodeselector-and-node-affinity.md)  |
| ↳ Expected Behavior | [44](notes/44-nodeselector-and-node-affinity.md)  |
| Key Considerations | [44](notes/44-nodeselector-and-node-affinity.md)  |
| Node Affinity, nodeSelector, and Taints & Tolerations | [44](notes/44-nodeselector-and-node-affinity.md)  |
| ↳ Node Affinity (nodeSelector, preferredDuringSchedulingIgnoredDuringExecution) | [44](notes/44-nodeselector-and-node-affinity.md)  |
| ↳ Behavior of nodeSelector | [44](notes/44-nodeselector-and-node-affinity.md)  |
| ↳ Taints & Tolerations (NoExecute Effect) | [44](notes/44-nodeselector-and-node-affinity.md)  |
| References | [44](notes/44-nodeselector-and-node-affinity.md)  |

| File | Used in |
|---|---|
| `na.yaml` | [44](notes/44-nodeselector-and-node-affinity.md)  |
| `na-preferred.yaml` | [44](notes/44-nodeselector-and-node-affinity.md)  |
| `ns.yaml` | [44](notes/44-nodeselector-and-node-affinity.md)  |

### Day 18 · Taints & Tolerations vs. Node Affinity | MASTER Pod Scheduling Control | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 18 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Introduction: Why Compare Taints & Tolerations with Node Affinity? | [45](notes/45-taints-vs-affinity.md)  |
| ↳ What We Observed with Taints & Tolerations | [45](notes/45-taints-vs-affinity.md)  |
| ↳ What We Observed with Node Affinity | [45](notes/45-taints-vs-affinity.md)  |
| ↳ Why Do We Need Both? | [45](notes/45-taints-vs-affinity.md)  |
| ↳ Key Takeaway | [45](notes/45-taints-vs-affinity.md)  |
| Scenario: Taints & Tolerations vs. Node Affinity in Action | [45](notes/45-taints-vs-affinity.md)  |
| ↳ Cluster Setup | [45](notes/45-taints-vs-affinity.md)  |
| ↳ Pod1: Using Only Tolerations | [45](notes/45-taints-vs-affinity.md)  |
| ↳ Pod2: Using Only Node Affinity | [45](notes/45-taints-vs-affinity.md)  |
| ↳ ⚠️ Important Caveat | [45](notes/45-taints-vs-affinity.md)  |
| ↳ Common Misconception | [45](notes/45-taints-vs-affinity.md)  |
| ↳ Pod3: No Tolerations, No Node Affinity | [45](notes/45-taints-vs-affinity.md)  |
| Final Solution: Using Both Taints & Tolerations + Node Affinity | [45](notes/45-taints-vs-affinity.md)  |
| Why Not Use Only Node Affinity? | [45](notes/45-taints-vs-affinity.md)  |
| ↳ Key Reasons to Use Taints & Tolerations Along with Node Affinity | [45](notes/45-taints-vs-affinity.md)  |
| Key Takeaways | [45](notes/45-taints-vs-affinity.md)  |
| Conclusion | [45](notes/45-taints-vs-affinity.md)  |
| References | [45](notes/45-taints-vs-affinity.md)  |

| Image | Used in |
|---|---|
| `18a.png` | [45](notes/45-taints-vs-affinity.md)  |

### Day 19 · MASTER Kubernetes Requests, Limits & LimitRange | MUST-KNOW Concepts | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 19 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Introduction | [41](notes/41-requests-limits-qos-quotas.md)  |
| ↳ Why Do We Need Requests and Limits? | [41](notes/41-requests-limits-qos-quotas.md)  |
| Benefits of Using Requests and Limits | [41](notes/41-requests-limits-qos-quotas.md)  |
| Applying Requests and Limits to Our Example | [41](notes/41-requests-limits-qos-quotas.md)  |
| What are Requests and Limits? | [41](notes/41-requests-limits-qos-quotas.md)  |
| Resource Types in Kubernetes | [41](notes/41-requests-limits-qos-quotas.md)  |
| How Requests and Limits Work | [41](notes/41-requests-limits-qos-quotas.md)  |
| ↳ 1️⃣ How Requests Affect Scheduling | [41](notes/41-requests-limits-qos-quotas.md)  |
| ↳ 2️⃣ How Limits Affect Running Containers | [41](notes/41-requests-limits-qos-quotas.md)  |
| What Happens When a Pod Exceeds Requests? | [41](notes/41-requests-limits-qos-quotas.md)  |
| What Happens When a Pod Exceeds Limits? | [41](notes/41-requests-limits-qos-quotas.md)  |
| ↳ CPU Limits: CPU Throttling | [41](notes/41-requests-limits-qos-quotas.md)  |
| ↳ Memory Limits: OOM Kills | [41](notes/41-requests-limits-qos-quotas.md)  |
| Monitoring Resource Utilization in Kubernetes | [41](notes/41-requests-limits-qos-quotas.md)  |
| How Kubernetes Collects Metrics | [41](notes/41-requests-limits-qos-quotas.md)  |
| Why Does kubectl top nodes Fail Without the Metrics Server? | [41](notes/41-requests-limits-qos-quotas.md)  |
| Installing the Metrics Server | [41](notes/41-requests-limits-qos-quotas.md)  |
| ↳ Step 1: Download the Metrics Server Manifest | [41](notes/41-requests-limits-qos-quotas.md)  |
| ↳ Step 2: Modify the Metrics Server Deployment for KIND | [41](notes/41-requests-limits-qos-quotas.md)  |
| ↳ Step 3: Apply the Metrics Server Manifest | [41](notes/41-requests-limits-qos-quotas.md)  |
| ↳ Step 4: Verify Metrics Server Installation | [41](notes/41-requests-limits-qos-quotas.md)  |
| ↳ Step 5: Ensure the Metrics Server is Running | [41](notes/41-requests-limits-qos-quotas.md)  |
| Demo: Memory Requests & Limits | [41](notes/41-requests-limits-qos-quotas.md)  |
| ↳ Step 1: Define a Pod with Memory Limits | [41](notes/41-requests-limits-qos-quotas.md)  |
| ↳ Step 2: Explanation | [41](notes/41-requests-limits-qos-quotas.md)  |
| ↳ Step 3: Increase Memory Usage Beyond Limit | [41](notes/41-requests-limits-qos-quotas.md)  |
| ↳ Step 4: Checking Assigned Requests and Limits | [41](notes/41-requests-limits-qos-quotas.md)  |
| Demo: CPU Requests & Limits | [41](notes/41-requests-limits-qos-quotas.md)  |
| ↳ Step 1: Define a Pod with CPU Limits | [41](notes/41-requests-limits-qos-quotas.md)  |
| ↳ Step 2: Explanation | [41](notes/41-requests-limits-qos-quotas.md)  |
| Default Requests and Limits (LimitRange) | [41](notes/41-requests-limits-qos-quotas.md)  |
| LimitRange: Setting Default Requests and Limits | [41](notes/41-requests-limits-qos-quotas.md)  |
| ↳ LimitRange YAML Example | [41](notes/41-requests-limits-qos-quotas.md)  |
| ↳ Explanation | [41](notes/41-requests-limits-qos-quotas.md)  |
| ↳ Applying the LimitRange | [41](notes/41-requests-limits-qos-quotas.md)  |
| ↳ Verifying the LimitRange | [41](notes/41-requests-limits-qos-quotas.md)  |
| Best Practices for Requests & Limits | [41](notes/41-requests-limits-qos-quotas.md)  |
| References | [41](notes/41-requests-limits-qos-quotas.md)  |

| Image | Used in |
|---|---|
| `19a.png` | [41](notes/41-requests-limits-qos-quotas.md)  |
| `19b.png` | [41](notes/41-requests-limits-qos-quotas.md)  |
| `19c.png` | [41](notes/41-requests-limits-qos-quotas.md)  |
| `19d.png` | [41](notes/41-requests-limits-qos-quotas.md)  |

| File | Used in |
|---|---|
| `components.yaml` | [41](notes/41-requests-limits-qos-quotas.md)  |
| `cpu.yaml` | [41](notes/41-requests-limits-qos-quotas.md)  |
| `limitrange.yaml` | [41](notes/41-requests-limits-qos-quotas.md)  |
| `memory.yaml` | [41](notes/41-requests-limits-qos-quotas.md)  |

### Day 20 · MASTER Kubernetes Autoscaling | HPA & VPA Explained with DEMO | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 20 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| 📑 Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| What is Scaling? | [48](notes/48-metrics-server-and-hpa.md)  |
| Types of Scaling | [48](notes/48-metrics-server-and-hpa.md)  |
| ↳ 1. Horizontal Scaling (Scaling Out/In) | [48](notes/48-metrics-server-and-hpa.md)  |
| ↳ 2. Vertical Scaling (Scaling Up/Down) | [49](notes/49-vpa-inplace-resize-node-autoscaling.md)  |
| Scaling Approaches | [48](notes/48-metrics-server-and-hpa.md)  |
| ↳ Horizontal Scaling: | [48](notes/48-metrics-server-and-hpa.md)  |
| ↳ Scaling Worker Nodes (Horizontally): | [48](notes/48-metrics-server-and-hpa.md)  |
| ↳ Vertical Scaling: | [49](notes/49-vpa-inplace-resize-node-autoscaling.md)  |
| ↳ Worker Nodes & Vertical Scaling: | [49](notes/49-vpa-inplace-resize-node-autoscaling.md)  |
| Pod vs Worker Node Scaling Approaches | [48](notes/48-metrics-server-and-hpa.md)  |
| How HPA Works: | [48](notes/48-metrics-server-and-hpa.md)  |
| Supported Workloads: | [48](notes/48-metrics-server-and-hpa.md)  |
| Why Resource Requests Are Mandatory for HPA? | [48](notes/48-metrics-server-and-hpa.md)  |
| Metrics Used by HPA: | [48](notes/48-metrics-server-and-hpa.md)  |
| ↳ 1. Default Resource Metrics (CPU & Memory) | [48](notes/48-metrics-server-and-hpa.md)  |
| ↳ 2. Custom Metrics Support | [48](notes/48-metrics-server-and-hpa.md)  |
| Metrics Server: Prerequisite for HPA | [48](notes/48-metrics-server-and-hpa.md)  |
| Demo: Horizontal Pod Autoscaler (HPA) | [48](notes/48-metrics-server-and-hpa.md)  |
| ↳ Step 1️⃣: Create Nginx Deployment | [48](notes/48-metrics-server-and-hpa.md)  |
| ↳ Step 2️⃣: Create Kubernetes Service to Expose Pods | [48](notes/48-metrics-server-and-hpa.md)  |
| ↳ Step 3️⃣: Create HPA Object | [48](notes/48-metrics-server-and-hpa.md)  |
| ↳ Step 4️⃣: Check HPA Status | [48](notes/48-metrics-server-and-hpa.md)  |
| ↳ Step 5️⃣: Generate CPU Load Using BusyBox | [48](notes/48-metrics-server-and-hpa.md)  |
| ↳ Step 6️⃣: Observe Auto-Scaling | [48](notes/48-metrics-server-and-hpa.md)  |
| Did You Know? | [48](notes/48-metrics-server-and-hpa.md)  |
| Vertical Pod Autoscaler (VPA) | [49](notes/49-vpa-inplace-resize-node-autoscaling.md)  |
| ↳ Key Behavior: | [49](notes/49-vpa-inplace-resize-node-autoscaling.md)  |
| Why use VPA? | [49](notes/49-vpa-inplace-resize-node-autoscaling.md)  |
| VPA Operating Modes: | [49](notes/49-vpa-inplace-resize-node-autoscaling.md)  |
| Components of VPA: | [49](notes/49-vpa-inplace-resize-node-autoscaling.md)  |
| ↳ VPA as a Resource Recommendation Tool | [49](notes/49-vpa-inplace-resize-node-autoscaling.md)  |
| ↳ HPA vs VPA: Recommendations and Complementary Use | [49](notes/49-vpa-inplace-resize-node-autoscaling.md)  |
| ↳ HPA vs VPA: Important Distinctions | [49](notes/49-vpa-inplace-resize-node-autoscaling.md)  |
| Step 1️⃣ - Install VPA Components | [49](notes/49-vpa-inplace-resize-node-autoscaling.md)  |
| ↳ Commands: | [49](notes/49-vpa-inplace-resize-node-autoscaling.md)  |
| ↳ Optional Uninstall: | [49](notes/49-vpa-inplace-resize-node-autoscaling.md)  |
| Step 2️⃣ - Create Nginx Deployment & Service | [48](notes/48-metrics-server-and-hpa.md)  |
| ↳ Deployment YAML (nginx-vpa.yaml): | [48](notes/48-metrics-server-and-hpa.md)  |
| ↳ Apply the YAML: | [48](notes/48-metrics-server-and-hpa.md)  |
| Step 3️⃣ - Create VPA Object | [49](notes/49-vpa-inplace-resize-node-autoscaling.md)  |
| Step 4️⃣ - Check VPA Recommendations | [49](notes/49-vpa-inplace-resize-node-autoscaling.md)  |
| ↳ Explaining Key Output: | [49](notes/49-vpa-inplace-resize-node-autoscaling.md)  |
| ↳ Uncapped Target vs Target in VPA | [49](notes/49-vpa-inplace-resize-node-autoscaling.md)  |
| Step 5️⃣ - Observe VPA Actions | [49](notes/49-vpa-inplace-resize-node-autoscaling.md)  |
| ↳ 1️⃣ Describe VPA to Check Updates: | [49](notes/49-vpa-inplace-resize-node-autoscaling.md)  |
| ↳ 2️⃣ Check Events: | [49](notes/49-vpa-inplace-resize-node-autoscaling.md)  |
| References: | [48](notes/48-metrics-server-and-hpa.md)  |

| Image | Used in |
|---|---|
| `20a.png` | [48](notes/48-metrics-server-and-hpa.md)  |
| `20b.png` | [48](notes/48-metrics-server-and-hpa.md)  |
| `20c.png` | [48](notes/48-metrics-server-and-hpa.md)  |
| `20d.png` | [48](notes/48-metrics-server-and-hpa.md)  |

| File | Used in |
|---|---|
| `nginx-hpa.yaml` | [48](notes/48-metrics-server-and-hpa.md)  |
| `nginx-vpa.yaml` | [49](notes/49-vpa-inplace-resize-node-autoscaling.md)  |
| `vpa.yaml` | [49](notes/49-vpa-inplace-resize-node-autoscaling.md)  |

### Day 21 · Multi-Container Pods DEEP-DIVE | Init vs Sidecar vs Ambassador vs Adapter | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 21 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ↳ Introduction | [23](notes/23-multi-container-pods.md)  |
| ↳ Why Multi-Container Pods? | [23](notes/23-multi-container-pods.md)  |
| ↳ What Are Multi-Container Pods? | [23](notes/23-multi-container-pods.md)  |
| ↳ Shared Resources in Multi-Container Pods | [23](notes/23-multi-container-pods.md)  |
| ↳ Multi-Container Pod Patterns | [23](notes/23-multi-container-pods.md)  |
| Differences: Init Containers vs Sidecar, Ambassador, Adapter Containers | [23](notes/23-multi-container-pods.md)  |
| ↳ Conclusion | [23](notes/23-multi-container-pods.md)  |
| ↳ Demo: Init Container | [23](notes/23-multi-container-pods.md)  |
| ↳ Explanation: | [23](notes/23-multi-container-pods.md)  |
| ↳ How to Run This Demo: | [23](notes/23-multi-container-pods.md)  |
| Extended Demo: Two Init Containers (API + Service Check) | [23](notes/23-multi-container-pods.md)  |
| ↳ Explanation: | [23](notes/23-multi-container-pods.md)  |
| ↳ How to Run This Demo: | [23](notes/23-multi-container-pods.md)  |
| ↳ Key Takeaway: | [23](notes/23-multi-container-pods.md)  |
| ↳ Summary | [23](notes/23-multi-container-pods.md)  |
| ↳ Demo: Sidecar Container for API Health Check and Logging | [23](notes/23-multi-container-pods.md)  |
| ↳ Pod YAML Example | [23](notes/23-multi-container-pods.md)  |
| ↳ Explanation | [23](notes/23-multi-container-pods.md)  |
| ↳ How to Run This Demo | [23](notes/23-multi-container-pods.md)  |
| ↳ References: | [23](notes/23-multi-container-pods.md)  |

| Image | Used in |
|---|---|
| `21a.png` | [23](notes/23-multi-container-pods.md)  |
| `21b.png` | [23](notes/23-multi-container-pods.md)  |
| `21c.png` | [23](notes/23-multi-container-pods.md)  |

| File | Used in |
|---|---|
| `2-init-c.yaml` | [23](notes/23-multi-container-pods.md)  |
| `init-c.yaml` | [23](notes/23-multi-container-pods.md)  |
| `sidecar.yaml` | [23](notes/23-multi-container-pods.md)  |
| `svc.yaml` | [23](notes/23-multi-container-pods.md)  |

### Day 22 · Kubernetes Pod Termination, Restart Policies, Image Pull Policy, Lifecycle & Common Errors | CKA 2025

| Original section | New home |
|---|---|
| Video reference for Day 22 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Introduction | [19](notes/19-pod-lifecycle-restarts-and-termination.md)  |
| Pod Deletion and Termination Signals | [19](notes/19-pod-lifecycle-restarts-and-termination.md)  |
| ↳ What Happens When You Run kubectl delete pod? | [19](notes/19-pod-lifecycle-restarts-and-termination.md)  |
| ↳ Example: Graceful Shutdown of a Web Server | [19](notes/19-pod-lifecycle-restarts-and-termination.md)  |
| ↳ Force Delete a Pod | [19](notes/19-pod-lifecycle-restarts-and-termination.md)  |
| ↳ Important: | [19](notes/19-pod-lifecycle-restarts-and-termination.md)  |
| Restart Policies | [19](notes/19-pod-lifecycle-restarts-and-termination.md)  |
| ↳ 1. Always | [19](notes/19-pod-lifecycle-restarts-and-termination.md)  |
| ↳ 2. OnFailure | [19](notes/19-pod-lifecycle-restarts-and-termination.md)  |
| ↳ 3. Never | [19](notes/19-pod-lifecycle-restarts-and-termination.md)  |
| ↳ Summary Table: | [19](notes/19-pod-lifecycle-restarts-and-termination.md)  |
| Image Pull Policies | [20](notes/20-images-and-pull-policy.md)  |
| ↳ 1. Always | [20](notes/20-images-and-pull-policy.md)  |
| ↳ 2. IfNotPresent | [20](notes/20-images-and-pull-policy.md)  |
| ↳ 3. Never | [20](notes/20-images-and-pull-policy.md)  |
| ↳ Default Behavior | [20](notes/20-images-and-pull-policy.md)  |
| ↳ Which imagePullPolicy Should You Use? | [20](notes/20-images-and-pull-policy.md)  |
| ↳ Key Considerations | [20](notes/20-images-and-pull-policy.md)  |
| Pod Lifecycle Phases | [19](notes/19-pod-lifecycle-restarts-and-termination.md)  |
| ↳ How ContainerCreating Fits In: | [19](notes/19-pod-lifecycle-restarts-and-termination.md)  |
| ↳ Pod Status in Multi-Container Pods | [19](notes/19-pod-lifecycle-restarts-and-termination.md)  |
| ↳ Differentiation Between Succeeded and Completed | [19](notes/19-pod-lifecycle-restarts-and-termination.md)  |
| ↳ Summary Table | [19](notes/19-pod-lifecycle-restarts-and-termination.md)  |
| Demo 1: Observing Short-Lived Pod Lifecycle | [19](notes/19-pod-lifecycle-restarts-and-termination.md)  |
| ↳ Steps | [19](notes/19-pod-lifecycle-restarts-and-termination.md)  |
| ↳ Key Takeaway | [19](notes/19-pod-lifecycle-restarts-and-termination.md)  |
| Demo 2: Observing Longer Pod Lifecycle | [19](notes/19-pod-lifecycle-restarts-and-termination.md)  |
| ↳ Steps | [19](notes/19-pod-lifecycle-restarts-and-termination.md)  |
| ↳ Key Takeaway | [19](notes/19-pod-lifecycle-restarts-and-termination.md)  |
| ↳ Quick Debugging Workflow | [21](notes/21-debugging-pods.md)  |
| Pod Status Deep Dive | [19](notes/19-pod-lifecycle-restarts-and-termination.md)  |
| Debugging Common Pod Errors | [21](notes/21-debugging-pods.md)  |
| ↳ Step-by-Step Debugging Workflow: | [21](notes/21-debugging-pods.md)  |
| ↳ References: | [21](notes/21-debugging-pods.md)  |

| Image | Used in |
|---|---|
| `22a.png` | [19](notes/19-pod-lifecycle-restarts-and-termination.md)  |
| `22b.png` | [19](notes/19-pod-lifecycle-restarts-and-termination.md)  |
| `22c.png` | [20](notes/20-images-and-pull-policy.md)  |
| `22d.png` | [19](notes/19-pod-lifecycle-restarts-and-termination.md)  |

| File | Used in |
|---|---|
| `pod.yaml` | [19](notes/19-pod-lifecycle-restarts-and-termination.md)  |

### Day 23 · DEEP-DIVE into Kubernetes Health Probes | Readiness vs Liveness vs Startup | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 23 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ↳ Introduction | [22](notes/22-health-probes.md)  |
| ↳ Types of Health Probes | [22](notes/22-health-probes.md)  |
| ↳ Why Configure Both Readiness and Liveness Probes? | [22](notes/22-health-probes.md)  |
| ↳ Probe Timer Configuration Parameters | [22](notes/22-health-probes.md)  |
| ↳ Behavior of RP, LP, and Startup Probes with Multi-Container Pods | [22](notes/22-health-probes.md)  |
| ↳ Kubernetes Health Probes: Comprehensive Comparison | [22](notes/22-health-probes.md)  |
| ↳ Probe Mechanisms | [22](notes/22-health-probes.md)  |
| ↳ Conclusion | [22](notes/22-health-probes.md)  |
| Demo: Readiness Probe with Command Execution | [22](notes/22-health-probes.md)  |
| Explanation: | [22](notes/22-health-probes.md)  |
| How to Test: | [22](notes/22-health-probes.md)  |
| How to Recover: | [22](notes/22-health-probes.md)  |
| Demo: Liveness Probe with HTTP GET | [22](notes/22-health-probes.md)  |
| Demo: Startup Probe with TCP Socket | [22](notes/22-health-probes.md)  |
| ↳ Explanation: | [22](notes/22-health-probes.md)  |
| ↳ Key Demo Points: | [22](notes/22-health-probes.md)  |
| References: | [22](notes/22-health-probes.md)  |

| Image | Used in |
|---|---|
| `23a.png` | [22](notes/22-health-probes.md)  |
| `23b.png` | [22](notes/22-health-probes.md)  |
| `23c.png` | [22](notes/22-health-probes.md)  |

| File | Used in |
|---|---|
| `lp.yaml` | [22](notes/22-health-probes.md)  |
| `rp.yaml` | [22](notes/22-health-probes.md)  |
| `sp.yaml` | [22](notes/22-health-probes.md)  |
| `test.yaml.yml` | [22](notes/22-health-probes.md)  |

### Day 24 · Docker Volumes Explained | Foundation for Kubernetes Persistent Storage | CKA 2025 Course

| Original section | New home |
|---|---|
| Video reference for Day 24 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ↳ Introduction | [36](notes/36-docker-storage-recap.md)  |
| ↳ Understanding Storage in Docker | [36](notes/36-docker-storage-recap.md)  |
| ↳ Layers in a Dockerfile | [36](notes/36-docker-storage-recap.md)  |
| ↳ Example: Layers in Our Dockerfile | [36](notes/36-docker-storage-recap.md)  |
| ↳ How to Inspect Image Layers | [36](notes/36-docker-storage-recap.md)  |
| ↳ How Docker Containers Use Image Layers | [36](notes/36-docker-storage-recap.md)  |
| ↳ Storage Calculation Example | [36](notes/36-docker-storage-recap.md)  |
| ↳ Handling Data Loss in Writable Layers | [36](notes/36-docker-storage-recap.md)  |
| ↳ Need for Persistent Storage | [36](notes/36-docker-storage-recap.md)  |
| Storage in Docker: How Does It Work? | [36](notes/36-docker-storage-recap.md)  |
| ↳ 1. Storage Drivers | [36](notes/36-docker-storage-recap.md)  |
| ↳ 2. Volume Drivers | [36](notes/36-docker-storage-recap.md)  |
| How Docker Storage Works on macOS, Windows, and Linux | [36](notes/36-docker-storage-recap.md)  |
| ↳ Linux | [36](notes/36-docker-storage-recap.md)  |
| ↳ macOS and Windows | [36](notes/36-docker-storage-recap.md)  |
| ↳ Understanding Docker Volumes | [36](notes/36-docker-storage-recap.md)  |
| ↳ Where Are Docker Volumes Stored? | [36](notes/36-docker-storage-recap.md)  |
| Demo: With Persistent Storage (Docker Volume) | [36](notes/36-docker-storage-recap.md)  |
| ↳ Note for macOS and Windows Users | [36](notes/36-docker-storage-recap.md)  |
| ↳ Types of Docker Volumes: Which to Use When? | [36](notes/36-docker-storage-recap.md)  |
| ↳ Conclusion | [36](notes/36-docker-storage-recap.md)  |
| ↳ References | [36](notes/36-docker-storage-recap.md)  |

| Image | Used in |
|---|---|
| `24a.png` | [36](notes/36-docker-storage-recap.md)  |
| `24b.png` | [36](notes/36-docker-storage-recap.md)  |
| `24c.png` | [36](notes/36-docker-storage-recap.md)  |
| `24d.png` | [36](notes/36-docker-storage-recap.md)  |
| `24e.png` | [36](notes/36-docker-storage-recap.md)  |

### Day 25 · Kubernetes Core & Extensions | CNI, CSI, CRI, Add-Ons & Plugins Explained | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 25 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Kubernetes Core Architecture | [06](notes/06-the-cluster-at-a-glance.md)  |
| ↳ What is Kubernetes Core? | [06](notes/06-the-cluster-at-a-glance.md)  |
| ↳ 1️⃣ Control Plane Components | [07](notes/07-control-plane-components.md)  |
| ↳ 2️⃣ Node Components | [08](notes/08-node-components.md)  |
| Kubernetes Extensions: Beyond the Core | [09](notes/09-plugins-interfaces-and-add-ons.md)  |
| ↳ 1️⃣ Plugins | [09](notes/09-plugins-interfaces-and-add-ons.md)  |
| ↳ 2️⃣ Add-Ons | [09](notes/09-plugins-interfaces-and-add-ons.md)  |
| ↳ 3️⃣ Third-Party Extensions | [09](notes/09-plugins-interfaces-and-add-ons.md)  |
| Deep Dive into Kubernetes Interfaces | [06](notes/06-the-cluster-at-a-glance.md)  |
| ↳ Overview of Kubernetes Interfaces | [06](notes/06-the-cluster-at-a-glance.md)  |
| Why Kubernetes Uses a Plugin-Based Architecture? | [09](notes/09-plugins-interfaces-and-add-ons.md)  |
| Key Takeaways | [06](notes/06-the-cluster-at-a-glance.md)  |
| Conclusion | [06](notes/06-the-cluster-at-a-glance.md)  |
| References | [06](notes/06-the-cluster-at-a-glance.md)  |

| Image | Used in |
|---|---|
| `25a.png` | [06](notes/06-the-cluster-at-a-glance.md)  |
| `25b.png` | [09](notes/09-plugins-interfaces-and-add-ons.md)  |

### Day 26 · Kubernetes Volumes | Ephemeral Storage | emptyDir & downwardAPI DEMO | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 26 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Introduction | [37](notes/37-ephemeral-volumes.md)  |
| ↳ Ephemeral Storage | [37](notes/37-ephemeral-volumes.md)  |
| ↳ Conclusion | [37](notes/37-ephemeral-volumes.md)  |
| ↳ References | [37](notes/37-ephemeral-volumes.md)  |

| Image | Used in |
|---|---|
| `26a.png` | [37](notes/37-ephemeral-volumes.md)  |

| File | Used in |
|---|---|
| `downwardapi-1.yaml` | [37](notes/37-ephemeral-volumes.md)  |
| `downwardapi-2.yaml` | [37](notes/37-ephemeral-volumes.md)  |
| `emptyDir.yaml` | [37](notes/37-ephemeral-volumes.md)  |

### Day 27 · Kubernetes Volumes | Persistent Storage | PV, PVC, StorageClass, hostPath DEMO | CKA 2025

| Original section | New home |
|---|---|
| Video reference for Day 27 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Introduction | [38](notes/38-persistent-storage-pv-pvc.md)  |
| Evolution of Storage in Kubernetes: From In-Tree to CSI Drivers | [38](notes/38-persistent-storage-pv-pvc.md)  |
| 1. In-Tree Volume Plugins: The Legacy Model | [38](notes/38-persistent-storage-pv-pvc.md)  |
| ↳ Examples of In-Tree Plugins: | [38](notes/38-persistent-storage-pv-pvc.md)  |
| 2. Container Storage Interface (CSI): The Modern Standard | [38](notes/38-persistent-storage-pv-pvc.md)  |
| ↳ Key Benefits of CSI: | [38](notes/38-persistent-storage-pv-pvc.md)  |
| ↳ Examples of CSI Drivers: | [38](notes/38-persistent-storage-pv-pvc.md)  |
| Special Cases: hostPath and local Storage | [38](notes/38-persistent-storage-pv-pvc.md)  |
| ↳ hostPath | [38](notes/38-persistent-storage-pv-pvc.md)  |
| ↳ local Volumes | [38](notes/38-persistent-storage-pv-pvc.md)  |
| ↳ Persistent Storage | [38](notes/38-persistent-storage-pv-pvc.md)  |
| ↳ Understanding hostPath in KIND: How Storage Works Under the Hood | [38](notes/38-persistent-storage-pv-pvc.md)  |
| ↳ Key Takeaway | [38](notes/38-persistent-storage-pv-pvc.md)  |
| ↳ Why This Matters | [38](notes/38-persistent-storage-pv-pvc.md)  |
| ↳ Demo: hostPath | [38](notes/38-persistent-storage-pv-pvc.md)  |
| ↳ Persistent Volumes (PVs) & Persistent Volume Claims (PVCs) | [38](notes/38-persistent-storage-pv-pvc.md)  |
| ↳ Understanding Scope & Relationships of PV and PVC in Kubernetes | [38](notes/38-persistent-storage-pv-pvc.md)  |
| ↳ Why Is This Important? | [38](notes/38-persistent-storage-pv-pvc.md)  |
| ↳ 1-to-1 Binding Relationship Between PVC and PV | [38](notes/38-persistent-storage-pv-pvc.md)  |
| ↳ Additional Key Points | [38](notes/38-persistent-storage-pv-pvc.md)  |
| ↳ Example Scenario | [38](notes/38-persistent-storage-pv-pvc.md)  |
| ↳ Kubernetes Persistent Storage Flow (Manual Provisioning) | [38](notes/38-persistent-storage-pv-pvc.md)  |
| ↳ Important Notes | [38](notes/38-persistent-storage-pv-pvc.md)  |
| ↳ Explanation of Storage Types | [38](notes/38-persistent-storage-pv-pvc.md)  |
| ↳ Key Differences: Block Storage vs. File Storage | [38](notes/38-persistent-storage-pv-pvc.md)  |
| ↳ Reclaim Policies in Kubernetes | [38](notes/38-persistent-storage-pv-pvc.md)  |
| ↳ Choosing the Right Reclaim Policy | [38](notes/38-persistent-storage-pv-pvc.md)  |
| ↳ PVC and PV Binding Conditions | [38](notes/38-persistent-storage-pv-pvc.md)  |
| ↳ Summary Table: PVC and PV Binding Conditions | [38](notes/38-persistent-storage-pv-pvc.md)  |
| ↳ Example Table: PVC vs. PV Matching | [38](notes/38-persistent-storage-pv-pvc.md)  |
| ↳ Critical Note for KIND/Minikube Users | [38](notes/38-persistent-storage-pv-pvc.md)  |
| ↳ Demo: Persistent Volumes and PVCs with Reclaim Policy | [38](notes/38-persistent-storage-pv-pvc.md)  |
| ↳ Storage Classes & Dynamic Provisioning | [39](notes/39-storageclasses-dynamic-provisioning.md)  |
| ↳ Example Storage Classes | [39](notes/39-storageclasses-dynamic-provisioning.md)  |
| ↳ Dynamic Provisioning in Action | [39](notes/39-storageclasses-dynamic-provisioning.md)  |
| ↳ Demo: Storage Class | [39](notes/39-storageclasses-dynamic-provisioning.md)  |
| ↳ Key Takeaways | [38](notes/38-persistent-storage-pv-pvc.md)  |
| ↳ Conclusion | [38](notes/38-persistent-storage-pv-pvc.md)  |
| ↳ References | [38](notes/38-persistent-storage-pv-pvc.md)  |

| Image | Used in |
|---|---|
| `27a.png` | [38](notes/38-persistent-storage-pv-pvc.md)  |
| `27b.png` | [38](notes/38-persistent-storage-pv-pvc.md)  |
| `27c.png` | [38](notes/38-persistent-storage-pv-pvc.md)  |
| `27d.png` | [39](notes/39-storageclasses-dynamic-provisioning.md)  |
| `27e.png` | [38](notes/38-persistent-storage-pv-pvc.md)  |

| File | Used in |
|---|---|
| `hostpath.yaml` | [38](notes/38-persistent-storage-pv-pvc.md)  |
| `pod.yaml` | [38](notes/38-persistent-storage-pv-pvc.md)  |
| `pv.yaml` | [38](notes/38-persistent-storage-pv-pvc.md)  |
| `pvc.yaml` | [38](notes/38-persistent-storage-pv-pvc.md)  |
| `sc.yaml` | [38](notes/38-persistent-storage-pv-pvc.md) [39](notes/39-storageclasses-dynamic-provisioning.md)  |

### Day 28 · Kubernetes ConfigMaps & Secrets Explained | Production-Ready Skills | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 28 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ↳ Introduction | [34](notes/34-configmaps.md)  |
| ConfigMaps | [34](notes/34-configmaps.md)  |
| ↳ Why Do We Need ConfigMaps? | [34](notes/34-configmaps.md)  |
| ↳ What is a ConfigMap? | [34](notes/34-configmaps.md)  |
| ↳ Without ConfigMap: Hardcoded Environment Variables | [34](notes/34-configmaps.md)  |
| Demo 1: Using ConfigMap as Environment Variables | [34](notes/34-configmaps.md)  |
| ↳ Step 1: Create ConfigMap | [34](notes/34-configmaps.md)  |
| ↳ Step 2: Reference ConfigMap in Deployment | [34](notes/34-configmaps.md)  |
| ↳ Verification | [34](notes/34-configmaps.md)  |
| Demo 2: Using ConfigMap as a Configuration File (Volume Mount) | [34](notes/34-configmaps.md)  |
| ↳ Step 1: Modify ConfigMap to Include HTML File | [34](notes/34-configmaps.md)  |
| ↳ Step 2: Mount ConfigMap Volume in Deployment | [34](notes/34-configmaps.md)  |
| ↳ What if you *do* want to mount the ConfigMap as a single file? | [34](notes/34-configmaps.md)  |
| Step 3: Expose the Deployment using a NodePort Service | [34](notes/34-configmaps.md)  |
| ↳ NodePort Service Manifest | [34](notes/34-configmaps.md)  |
| Verification | [34](notes/34-configmaps.md)  |
| ↳ Best Practices for ConfigMap | [34](notes/34-configmaps.md)  |
| Kubernetes Secrets | [35](notes/35-secrets.md)  |
| ↳ Why Do We Need Secrets in Kubernetes? | [35](notes/35-secrets.md)  |
| ↳ What is a Kubernetes Secret? | [35](notes/35-secrets.md)  |
| ↳ Important Distinction: Encoding vs. Encryption | [35](notes/35-secrets.md)  |
| ↳ Encoding vs. Encryption | [35](notes/35-secrets.md)  |
| ↳ Demo 1: Injecting Secrets into a Pod | [35](notes/35-secrets.md)  |
| ↳ Step 1: Create the Secret | [35](notes/35-secrets.md)  |
| ↳ Step 2: Update the Deployment to Use the Secret | [35](notes/35-secrets.md)  |
| ↳ Step 3: Verify Secret Injection | [35](notes/35-secrets.md)  |
| ↳ Alternative: Mount Secret as Files | [35](notes/35-secrets.md)  |
| ↳ Best Practices for Using Kubernetes Secrets | [35](notes/35-secrets.md)  |
| ↳ Understanding Dynamic Updates with ConfigMaps and Secrets | [35](notes/35-secrets.md)  |
| Conclusion | [34](notes/34-configmaps.md)  |
| References | [34](notes/34-configmaps.md)  |

| File | Used in |
|---|---|
| `backend-2.yaml` | [35](notes/35-secrets.md)  |
| `cm-1.yaml` | [34](notes/34-configmaps.md)  |
| `cm-2.yaml` | [34](notes/34-configmaps.md)  |
| `ev.yaml` | [34](notes/34-configmaps.md)  |
| `frontend-cm-1.yaml` | [34](notes/34-configmaps.md)  |
| `frontend-cm-2.yaml` | [34](notes/34-configmaps.md)  |
| `frontend-secret-1.yaml` | [35](notes/35-secrets.md)  |
| `frontend-secret-2.yaml` | [35](notes/35-secrets.md)  |
| `secret-1.yaml` | [35](notes/35-secrets.md)  |

### Day 29 · MASTER DaemonSet, Job & CronJob in Kubernetes  with Detailed Demos | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 29 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Understanding DaemonSets in Kubernetes | [27](notes/27-daemonsets.md)  |
| ↳ Use Cases for DaemonSets | [27](notes/27-daemonsets.md)  |
| ↳ DaemonSets We Have Already Seen: kube-proxy | [27](notes/27-daemonsets.md)  |
| ↳ DaemonSets for CNIs and CSIs | [27](notes/27-daemonsets.md)  |
| ↳ DaemonSets and Control Plane Nodes | [27](notes/27-daemonsets.md)  |
| ↳ Key Takeaways | [27](notes/27-daemonsets.md)  |
| Demo: DaemonSet - Deploying a Dummy Logging Agent | [27](notes/27-daemonsets.md)  |
| ↳ Step 1: Create a New Namespace for Logging | [27](notes/27-daemonsets.md)  |
| ↳ Step 2: Switch the Context to the New Namespace | [27](notes/27-daemonsets.md)  |
| ↳ Step 3: Apply the DaemonSet Manifest | [27](notes/27-daemonsets.md)  |
| ↳ Step 4: Verify the DaemonSet | [27](notes/27-daemonsets.md)  |
| ↳ Bonus Exercise: Observe DaemonSet Pod Re-Creation | [27](notes/27-daemonsets.md)  |
| Understanding Jobs and CronJobs in Kubernetes | [28](notes/28-jobs-and-cronjobs.md)  |
| ↳ What is a Job in Kubernetes? | [28](notes/28-jobs-and-cronjobs.md)  |
| ↳ Job Demo: Running a One-Time Task | [28](notes/28-jobs-and-cronjobs.md)  |
| What is a CronJob in Kubernetes? | [28](notes/28-jobs-and-cronjobs.md)  |
| Understanding Cron Expression Fields | [27](notes/27-daemonsets.md)  |
| Quick Visual Cheat-Sheet | [27](notes/27-daemonsets.md)  |
| ↳ Common Cron Expressions for Kubernetes CronJobs | [28](notes/28-jobs-and-cronjobs.md)  |
| ↳ Demo: CronJob - Running a Scheduled Task | [28](notes/28-jobs-and-cronjobs.md)  |
| Deployment vs DaemonSet vs Job vs CronJob | [28](notes/28-jobs-and-cronjobs.md)  |
| ↳ References | [28](notes/28-jobs-and-cronjobs.md)  |

| Image | Used in |
|---|---|
| `29a.png` | [27](notes/27-daemonsets.md)  |
| `29b.png` | [28](notes/28-jobs-and-cronjobs.md)  |
| `29c.png` | [28](notes/28-jobs-and-cronjobs.md)  |

| File | Used in |
|---|---|
| `cronjob.yaml` | [28](notes/28-jobs-and-cronjobs.md)  |
| `ds.yaml` | [27](notes/27-daemonsets.md)  |
| `job.yaml` | [28](notes/28-jobs-and-cronjobs.md)  |

### Day 30 · How HTTPS & SSH Work | What is Encryption? | Symmetric & Asymmetric Encryption | CKA 2025

| Original section | New home |
|---|---|
| Video reference for Day 30 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| What is Encryption? Why Do We Need It? | [50](notes/50-encryption-basics.md)  |
| Two Main Types of Encryption | [50](notes/50-encryption-basics.md)  |
| ↳ 1. Symmetric Encryption (Shared Key Encryption) | [50](notes/50-encryption-basics.md)  |
| ↳ 2. Asymmetric Encryption (Public Key Cryptography) | [50](notes/50-encryption-basics.md)  |
| Summary Table | [50](notes/50-encryption-basics.md)  |
| ↳ Symmetric and Asymmetric Encryption: Roles and Functions | [50](notes/50-encryption-basics.md)  |
| Encryption Approaches: In-Transit vs At-Rest | [50](notes/50-encryption-basics.md)  |
| ↳ 1. Encryption In-Transit | [50](notes/50-encryption-basics.md)  |
| ↳ 2. Encryption At-Rest | [50](notes/50-encryption-basics.md)  |
| ↳ Quick Comparison: | [50](notes/50-encryption-basics.md)  |
| Understanding Symmetric and Asymmetric Encryption Through Real-Life Scenarios | [50](notes/50-encryption-basics.md)  |
| ↳ Scenario 1: Symmetric Encryption (Disk Encryption) | [50](notes/50-encryption-basics.md)  |
| ↳ Scenario 2: Symmetric Encryption Alone Isn't Enough for Internet Communication | [50](notes/50-encryption-basics.md)  |
| ↳ Scenario 3: Asymmetric Encryption (SSH into EC2 Instance) | [50](notes/50-encryption-basics.md)  |
| ↳ Scenario 4: Combining Symmetric + Asymmetric (TLS Handshake) | [50](notes/50-encryption-basics.md)  |
| ↳ Key Takeaways: | [50](notes/50-encryption-basics.md)  |
| Conclusion | [50](notes/50-encryption-basics.md)  |

| Image | Used in |
|---|---|
| `30a.png` | [50](notes/50-encryption-basics.md)  |
| `30b.png` | [50](notes/50-encryption-basics.md)  |
| `30c.png` | [50](notes/50-encryption-basics.md)  |
| `30d.png` | [50](notes/50-encryption-basics.md)  |
| `30e.png` | [50](notes/50-encryption-basics.md)  |
| `30f.png` | [50](notes/50-encryption-basics.md)  |

### Day 31 · TLS in Kubernetes MASTERCLASS | PART 1 | Mutual Auth (SSH), TLS 1.3, Types of CAs | CKA 2025

| Original section | New home |
|---|---|
| Video reference for Day 31 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ↳ Pre-Requisites for Day 31 | [51](notes/51-certificates-and-cas.md)  |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ↳ Introduction | [51](notes/51-certificates-and-cas.md)  |
| ↳ Client and Server – A Refresher | [51](notes/51-certificates-and-cas.md)  |
| Public Key Cryptography | [51](notes/51-certificates-and-cas.md)  |
| ↳ Public Key Cryptography in DevOps: Focus on SSH & HTTPS | [51](notes/51-certificates-and-cas.md)  |
| ↳ 1. Authentication (Identity Verification): | [51](notes/51-certificates-and-cas.md)  |
| ↳ Order of Authentication in Mutual Authentication (SSH or mTLS) | [51](notes/51-certificates-and-cas.md)  |
| ↳ 2. Secure Key Exchange (Foundation for Encryption): | [51](notes/51-certificates-and-cas.md)  |
| ↳ Types of TLS Certificate Authorities (CA): Public, Private, and Self-Signed | [51](notes/51-certificates-and-cas.md)  |
| ↳ Public CA | [51](notes/51-certificates-and-cas.md)  |
| ↳ Private CA | [51](notes/51-certificates-and-cas.md)  |
| ↳ Self-Signed Certificate | [51](notes/51-certificates-and-cas.md)  |
| ↳ Conclusion | [51](notes/51-certificates-and-cas.md)  |

| Image | Used in |
|---|---|
| `31-1.png` | [51](notes/51-certificates-and-cas.md)  |
| `31-2.png` | [51](notes/51-certificates-and-cas.md)  |
| `31-3.png` | [51](notes/51-certificates-and-cas.md)  |
| `31-4.png` | [51](notes/51-certificates-and-cas.md)  |
| `31-5.png` | [51](notes/51-certificates-and-cas.md)  |
| `31-6.png` | [51](notes/51-certificates-and-cas.md)  |
| `31-7.png` | [51](notes/51-certificates-and-cas.md)  |
| `31-8.png` | [51](notes/51-certificates-and-cas.md)  |

### Day 32 · TLS in Kubernetes MASTERCLASS | PART 2 | mTLS , kubeconfig & Kubernetes Context | CKA 2025

| Original section | New home |
|---|---|
| Video reference for Day 32 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ↳ Pre-Requisites for Day 32 | [52](notes/52-kubeconfig-deep-dive-and-mtls.md)  |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Introduction | [52](notes/52-kubeconfig-deep-dive-and-mtls.md)  |
| Kubeconfig and Kubernetes Context | [52](notes/52-kubeconfig-deep-dive-and-mtls.md) [12](notes/12-kubectl-kubeconfig-and-contexts.md)  |
| ↳ What is a Kubeconfig File and Why Do We Need It? | [52](notes/52-kubeconfig-deep-dive-and-mtls.md) [12](notes/12-kubectl-kubeconfig-and-contexts.md)  |
| ↳ What If the Kubeconfig File Were Not There? | [52](notes/52-kubeconfig-deep-dive-and-mtls.md) [12](notes/12-kubectl-kubeconfig-and-contexts.md)  |
| ↳ Kubernetes Context | [52](notes/52-kubeconfig-deep-dive-and-mtls.md) [12](notes/12-kubectl-kubeconfig-and-contexts.md)  |
| Scenario: | [52](notes/52-kubeconfig-deep-dive-and-mtls.md) [12](notes/12-kubectl-kubeconfig-and-contexts.md)  |
| How Kubernetes Contexts Work: | [52](notes/52-kubeconfig-deep-dive-and-mtls.md) [12](notes/12-kubectl-kubeconfig-and-contexts.md)  |
| ↳ 1. Configuring Contexts: | [52](notes/52-kubeconfig-deep-dive-and-mtls.md) [12](notes/12-kubectl-kubeconfig-and-contexts.md)  |
| ↳ 2. Switching Between Contexts: | [52](notes/52-kubeconfig-deep-dive-and-mtls.md) [12](notes/12-kubectl-kubeconfig-and-contexts.md)  |
| Example kubeconfig File: | [52](notes/52-kubeconfig-deep-dive-and-mtls.md) [12](notes/12-kubectl-kubeconfig-and-contexts.md)  |
| ↳ Viewing & Managing Your kubeconfig | [52](notes/52-kubeconfig-deep-dive-and-mtls.md) [12](notes/12-kubectl-kubeconfig-and-contexts.md)  |
| ↳ Common kubectl config Commands (Compact View) | [52](notes/52-kubeconfig-deep-dive-and-mtls.md) [12](notes/12-kubectl-kubeconfig-and-contexts.md)  |
| ↳ Multiple Kubeconfig Files | [52](notes/52-kubeconfig-deep-dive-and-mtls.md) [12](notes/12-kubectl-kubeconfig-and-contexts.md)  |
| ↳ Using a Custom Kubeconfig as Default via KUBECONFIG | [52](notes/52-kubeconfig-deep-dive-and-mtls.md) [12](notes/12-kubectl-kubeconfig-and-contexts.md)  |
| ↳ Adding Entries to a Kubeconfig File | [52](notes/52-kubeconfig-deep-dive-and-mtls.md) [12](notes/12-kubectl-kubeconfig-and-contexts.md)  |
| Mutual TLS (mTLS) | [52](notes/52-kubeconfig-deep-dive-and-mtls.md)  |
| Conclusion | [52](notes/52-kubeconfig-deep-dive-and-mtls.md)  |
| References | [52](notes/52-kubeconfig-deep-dive-and-mtls.md)  |

| Image | Used in |
|---|---|
| `31-10.png` | [52](notes/52-kubeconfig-deep-dive-and-mtls.md)  |
| `31-12.png` | [52](notes/52-kubeconfig-deep-dive-and-mtls.md)  |
| `31-9.png` | [52](notes/52-kubeconfig-deep-dive-and-mtls.md)  |

### Day 33 · TLS in Kubernetes MASTERCLASS | PART 3 | Private CAs, TLS Auth Flow, mTLS in Kubernetes Cluster | CKA 2025

| Original section | New home |
|---|---|
| Video reference for Day 33 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ↳ Pre-Requisites for Day 33 | [53](notes/53-tls-inside-the-cluster.md)  |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Introduction | [53](notes/53-tls-inside-the-cluster.md)  |
| ↳ Private CAs in Kubernetes Clusters | [53](notes/53-tls-inside-the-cluster.md)  |
| ↳ Kubernetes Components as Clients and Servers | [53](notes/53-tls-inside-the-cluster.md)  |
| ↳ Client (Initiates a Request) | [53](notes/53-tls-inside-the-cluster.md)  |
| ↳ Server (Responds to the Request) | [53](notes/53-tls-inside-the-cluster.md)  |
| Understanding TLS in Kubernetes: No More Memorizing File Paths | [53](notes/53-tls-inside-the-cluster.md)  |
| Example 1: mTLS Between Scheduler (Client) and API Server (Server) | [53](notes/53-tls-inside-the-cluster.md)  |
| ↳ Key Principle: All Clients Use a Kubeconfig File to Connect to the API Server | [53](notes/53-tls-inside-the-cluster.md)  |
| Key TLS and Configuration File Locations (Control Plane & Nodes) | [53](notes/53-tls-inside-the-cluster.md)  |
| SERVER-SIDE: How the API Server Presents Its Certificate | [53](notes/53-tls-inside-the-cluster.md)  |
| ↳ Step 1: Who’s the Server? | [53](notes/53-tls-inside-the-cluster.md)  |
| ↳ Step 2: How Does the Server Present a Certificate? | [53](notes/53-tls-inside-the-cluster.md)  |
| ↳ Step 3: How Does the Scheduler Trust the API Server’s Certificate? | [53](notes/53-tls-inside-the-cluster.md)  |
| CLIENT-SIDE: How the Scheduler Authenticates to the API Server | [53](notes/53-tls-inside-the-cluster.md)  |
| ↳ Step 1: How Does the Scheduler Authenticate Itself? | [53](notes/53-tls-inside-the-cluster.md)  |
| ↳ Step 2: How Does the API Server Verify the Scheduler? | [53](notes/53-tls-inside-the-cluster.md)  |
| Conclusion: Focus on Config Files, Not Hardcoded Paths | [53](notes/53-tls-inside-the-cluster.md)  |
| ↳ Summary Table | [53](notes/53-tls-inside-the-cluster.md)  |
| Example 2: mTLS Between API Server (Client) and etcd (Server) | [53](notes/53-tls-inside-the-cluster.md)  |
| SERVER-SIDE: etcd Presents Its Certificate to API Server | [53](notes/53-tls-inside-the-cluster.md)  |
| ↳ Step 1: Know Who’s the Server | [53](notes/53-tls-inside-the-cluster.md)  |
| ↳ Step 2: Where Is etcd's Certificate Defined? | [53](notes/53-tls-inside-the-cluster.md)  |
| ↳ Step 3: How Does the API Server Verify etcd's Certificate? | [53](notes/53-tls-inside-the-cluster.md)  |
| CLIENT-SIDE: API Server Authenticates to etcd | [53](notes/53-tls-inside-the-cluster.md)  |
| ↳ Step 1: Where Does API Server Specify TLS Credentials? | [53](notes/53-tls-inside-the-cluster.md)  |
| ↳ Step 2: How Does etcd Verify the API Server's Certificate? | [53](notes/53-tls-inside-the-cluster.md)  |
| ↳ How Mutual TLS (mTLS) Works Here | [53](notes/53-tls-inside-the-cluster.md)  |
| ↳ Takeaway: Focus on Flags, Not File Paths | [53](notes/53-tls-inside-the-cluster.md)  |
| ↳ Summary Table | [53](notes/53-tls-inside-the-cluster.md)  |
| Example 3: mTLS Between API Server (Client) and Kubelet (Server) | [53](notes/53-tls-inside-the-cluster.md)  |
| Who is the Client and Who is the Server? | [53](notes/53-tls-inside-the-cluster.md)  |
| SERVER-SIDE: How the Kubelet Presents Its Certificate | [53](notes/53-tls-inside-the-cluster.md)  |
| ↳ Step 1: Locate the Kubelet Process | [53](notes/53-tls-inside-the-cluster.md)  |
| ↳ Step 2: Inspect the Server Certificate | [53](notes/53-tls-inside-the-cluster.md)  |
| ↳ How Did the Kubelet Get This Certificate? | [53](notes/53-tls-inside-the-cluster.md)  |
| ↳ Step 3: How the API Server Verifies the Kubelet (or Doesn’t) | [53](notes/53-tls-inside-the-cluster.md)  |
| CLIENT-SIDE: How the API Server Presents Its Own Certificate to the Kubelet | [53](notes/53-tls-inside-the-cluster.md)  |
| ↳ How the Kubelet Verifies the API Server's Certificate | [53](notes/53-tls-inside-the-cluster.md)  |
| Summary: mTLS Between API Server and Kubelet | [53](notes/53-tls-inside-the-cluster.md)  |
| Recap: How mTLS Happens Here | [53](notes/53-tls-inside-the-cluster.md)  |
| Tasks for You | [53](notes/53-tls-inside-the-cluster.md)  |
| Conclusion | [53](notes/53-tls-inside-the-cluster.md)  |
| References | [53](notes/53-tls-inside-the-cluster.md)  |

| Image | Used in |
|---|---|
| `33a.png` | [53](notes/53-tls-inside-the-cluster.md)  |
| `33b.png` | [53](notes/53-tls-inside-the-cluster.md)  |
| `33c.png` | [53](notes/53-tls-inside-the-cluster.md)  |

### Day 34 · TLS in Kubernetes MASTERCLASS | PART 4 | CSR, External User Access to K8S Cluster | CKA 2025

| Original section | New home |
|---|---|
| Video reference for Day 34 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ↳ Pre-Requisites for Day 34 | [54](notes/54-authentication.md)  |
| Introduction | [54](notes/54-authentication.md)  |
| ↳ Granting Cluster Access to a New User (Seema) using Certificates and RBAC | [57](notes/57-rbac.md)  |
| ↳ Conclusion | [54](notes/54-authentication.md)  |
| ↳ References | [54](notes/54-authentication.md)  |

| Image | Used in |
|---|---|
| `34a.png` | [54](notes/54-authentication.md)  |

### Day 35 · MASTER Kubernetes Authorization Modes and Kubernetes API | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 35 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Introduction | [14](notes/14-the-kubernetes-api.md)  |
| Kubernetes API: The Engine Behind Everything | [14](notes/14-the-kubernetes-api.md)  |
| ↳ What Is the Kubernetes API? | [14](notes/14-the-kubernetes-api.md)  |
| ↳ Key Pointers | [14](notes/14-the-kubernetes-api.md)  |
| 1. Kubernetes API Endpoints Overview | [14](notes/14-the-kubernetes-api.md)  |
| ↳ API Endpoints: Resource vs Non-Resource | [14](notes/14-the-kubernetes-api.md)  |
| API Groups: Organizing the Kubernetes API | [14](notes/14-the-kubernetes-api.md)  |
| ↳ Why the Core Group Name Isn’t Shown in Manifests | [14](notes/14-the-kubernetes-api.md)  |
| ↳ Accessing Unauthenticated and Authenticated Kubernetes API Endpoints via curl | [14](notes/14-the-kubernetes-api.md)  |
| Using kubectl proxy to Simplify API Access | [14](notes/14-the-kubernetes-api.md)  |
| ↳ What kubectl proxy Does | [14](notes/14-the-kubernetes-api.md)  |
| ↳ Example: Accessing the API via the Proxy | [14](notes/14-the-kubernetes-api.md)  |
| Bonus: Understanding API Paths | [14](notes/14-the-kubernetes-api.md)  |
| ↳ Key Points: | [14](notes/14-the-kubernetes-api.md)  |
| Authorization in Kubernetes | [56](notes/56-authorization-modes.md)  |
| High-Level Flow of Authorization | [56](notes/56-authorization-modes.md)  |
| Authorization is Context-Aware | [56](notes/56-authorization-modes.md)  |
| Types of Authorization Modes in Kubernetes | [56](notes/56-authorization-modes.md)  |
| ↳ 1. RBAC (Role-Based Access Control) | [56](notes/56-authorization-modes.md)  |
| ↳ 2. ABAC (Attribute-Based Access Control) | [56](notes/56-authorization-modes.md)  |
| ↳ 3. Webhook Authorization | [56](notes/56-authorization-modes.md)  |
| ↳ 4. Node Authorization | [56](notes/56-authorization-modes.md)  |
| ↳ Key Points: | [56](notes/56-authorization-modes.md)  |
| ↳ 5. AlwaysAllow / AlwaysDeny Authorization Modes | [56](notes/56-authorization-modes.md)  |
| How to Check and Change the Authorization Mode in Kubernetes | [56](notes/56-authorization-modes.md)  |
| ↳ 1. Authorization Happens in the API Server | [56](notes/56-authorization-modes.md)  |
| ↳ 2. Where to Find the API Server Configuration | [56](notes/56-authorization-modes.md)  |
| ↳ 3. Multiple Authorization Modes Can Be Used Together | [56](notes/56-authorization-modes.md)  |
| ↳ 4. How to Change Authorization Modes | [56](notes/56-authorization-modes.md)  |
| ↳ 5. Special Authorization Modes: AlwaysAllow and AlwaysDeny | [56](notes/56-authorization-modes.md)  |
| ↳ Key Points: | [56](notes/56-authorization-modes.md)  |
| ↳ Combining RBAC and Webhook Authorization | [56](notes/56-authorization-modes.md)  |
| Authorization Flow When Node, Webhook Authorizer, and RBAC Are Used Together | [56](notes/56-authorization-modes.md)  |
| ↳ Clarification: OPA Can Be Used in Two Ways | [56](notes/56-authorization-modes.md)  |
| ↳ Scenario 1: Using Node + Webhook Authorizer Only | [56](notes/56-authorization-modes.md)  |
| ↳ Scenario 2: Node + Webhook + RBAC (Layered) | [56](notes/56-authorization-modes.md)  |
| ↳ ✅ Summary Table | [56](notes/56-authorization-modes.md)  |
| Conclusion | [56](notes/56-authorization-modes.md)  |
| References | [56](notes/56-authorization-modes.md)  |

| Image | Used in |
|---|---|
| `35a.png` | [56](notes/56-authorization-modes.md)  |
| `35b.png` | [56](notes/56-authorization-modes.md)  |
| `35c.png` | [14](notes/14-the-kubernetes-api.md)  |

### Day 36 · Deep Dive into Kubernetes RBAC Authorization | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 36 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Pre-requisite | [57](notes/57-rbac.md)  |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Introduction | [57](notes/57-rbac.md)  |
| Namespaced vs Cluster-Scoped Resources and RBAC Verbs | [57](notes/57-rbac.md)  |
| What is RBAC? | [57](notes/57-rbac.md)  |
| ↳ Setting Up the dev Namespace as Default | [57](notes/57-rbac.md)  |
| Starting with Role and RoleBinding (Namespace-Scoped Permissions) | [57](notes/57-rbac.md)  |
| ↳ Step 1. Define a Role in the dev Namespace | [57](notes/57-rbac.md)  |
| ↳ Step 2. Bind the Role to Seema (and a Group) using RoleBinding | [57](notes/57-rbac.md)  |
| What Happens When Seema Runs a Command? | [57](notes/57-rbac.md)  |
| ↳ Key Pointers: | [57](notes/57-rbac.md)  |
| ClusterRole and ClusterRoleBinding (Cluster-Scoped Permissions) | [57](notes/57-rbac.md)  |
| ↳ Step 1: Define a ClusterRole (node-reader) | [57](notes/57-rbac.md)  |
| ↳ Step 2: Bind ClusterRole to Seema using ClusterRoleBinding | [57](notes/57-rbac.md)  |
| ↳ Bonus Tip: What about ServiceAccounts? | [57](notes/57-rbac.md)  |
| Verify Seema’s Effective Access | [57](notes/57-rbac.md)  |
| Extra Insight: ClusterRoles Can Be Used in Namespaces Too! | [57](notes/57-rbac.md)  |
| ↳ Step 1: Define a ClusterRole (elevated-workload-editor) | [57](notes/57-rbac.md)  |
| ↳ Step 2: Bind ClusterRole to a user in a specific namespace (prod) | [57](notes/57-rbac.md)  |
| Quick Recap: RBAC Building Blocks | [57](notes/57-rbac.md)  |
| Real-World Advice & Best Practices | [57](notes/57-rbac.md)  |
| Conclusion | [57](notes/57-rbac.md)  |
| References | [57](notes/57-rbac.md)  |

| File | Used in |
|---|---|
| `1-role-rb.yaml` | [57](notes/57-rbac.md)  |
| `2-cr-crb.yaml` | [57](notes/57-rbac.md)  |
| `3-cr-rb.yaml` | [57](notes/57-rbac.md)  |

### Day 37 · MASTER Kubernetes Service Accounts & Authentication | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 37 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Introduction | [54](notes/54-authentication.md)  |
| Authentication in Kubernetes | [54](notes/54-authentication.md)  |
| ↳ 1. Static Password File (Basic Auth) | [54](notes/54-authentication.md)  |
| ↳ 2. Token-Based Authentication | [54](notes/54-authentication.md)  |
| ↳ 3. Certificates (Client TLS) | [54](notes/54-authentication.md)  |
| ↳ 4. External Identity Providers | [54](notes/54-authentication.md)  |
| ↳ Understanding Who Interacts with a Kubernetes Cluster | [55](notes/55-serviceaccounts-and-tokens.md)  |
| ↳ Service Accounts in Kubernetes | [54](notes/54-authentication.md)  |
| ↳ Why Are Service Accounts Necessary? | [54](notes/54-authentication.md)  |
| ↳ Should You Use a Normal User Account for Automation? | [54](notes/54-authentication.md)  |
| ↳ Managing Service Account Permissions | [54](notes/54-authentication.md)  |
| ↳ Following the Principle of Least Privilege | [54](notes/54-authentication.md)  |
| ↳ How Do ServiceAccounts Authenticate with the Cluster? | [54](notes/54-authentication.md)  |
| ↳ Associating a ServiceAccount with a Pod | [54](notes/54-authentication.md)  |
| ↳ Why Does This Matter? | [54](notes/54-authentication.md)  |
| ↳ Evolution of ServiceAccount Tokens | [54](notes/54-authentication.md)  |
| ↳ Where Is the ServiceAccount Token Mounted? | [54](notes/54-authentication.md)  |
| ↳ How a Pod Authenticates with the API Server Using a ServiceAccount | [54](notes/54-authentication.md)  |
| Types of ServiceAccount Tokens in Kubernetes | [55](notes/55-serviceaccounts-and-tokens.md)  |
| ↳ 1. Bound ServiceAccount Tokens (aka Projected Tokens via TokenRequest API) | [55](notes/55-serviceaccounts-and-tokens.md)  |
| ↳ 2. Manually Requested Tokens (TokenRequest API for external tools) | [55](notes/55-serviceaccounts-and-tokens.md)  |
| ↳ 3. Legacy (Perpetual) Tokens via Secrets | [55](notes/55-serviceaccounts-and-tokens.md)  |
| ↳ ✅ Summary Table: | [55](notes/55-serviceaccounts-and-tokens.md)  |
| Non-Human Access to Kubernetes: A Detailed Breakdown | [55](notes/55-serviceaccounts-and-tokens.md)  |
| ↳ Internal to the Cluster | [55](notes/55-serviceaccounts-and-tokens.md)  |
| ↳ Authentication Methods Used by Internal Workloads | [55](notes/55-serviceaccounts-and-tokens.md)  |
| ↳ External to the Cluster | [55](notes/55-serviceaccounts-and-tokens.md)  |
| ↳ Authentication Methods Used by External Tools | [55](notes/55-serviceaccounts-and-tokens.md)  |
| ↳ TokenRequest API: Authentication Flow | [55](notes/55-serviceaccounts-and-tokens.md)  |
| ↳ Clarifying the Agent-Controller Model | [55](notes/55-serviceaccounts-and-tokens.md)  |
| Access Patterns Summary Table | [55](notes/55-serviceaccounts-and-tokens.md)  |
| Demo: Creating and Using a ServiceAccount for Jenkins | [55](notes/55-serviceaccounts-and-tokens.md)  |
| ↳ Step 1: Create a Service Account for Jenkins | [55](notes/55-serviceaccounts-and-tokens.md)  |
| ↳ Step 2: Create ClusterRole and ClusterRoleBinding for Jenkins SA | [55](notes/55-serviceaccounts-and-tokens.md)  |
| ↳ Step 3: Generate a Manual Long-Lived Token (Deprecated Approach) | [55](notes/55-serviceaccounts-and-tokens.md)  |
| ↳ Step 4: Jenkins Requests a Token Using the TokenRequest API (Recommended Approach) | [55](notes/55-serviceaccounts-and-tokens.md)  |
| ↳ Step 5: Jenkins Uses the Token to Authenticate | [55](notes/55-serviceaccounts-and-tokens.md)  |
| ↳ Key Takeaways | [55](notes/55-serviceaccounts-and-tokens.md)  |
| Conclusion | [55](notes/55-serviceaccounts-and-tokens.md)  |
| References | [55](notes/55-serviceaccounts-and-tokens.md)  |

| Image | Used in |
|---|---|
| `37a.png` | [54](notes/54-authentication.md)  |
| `37b.png` | [55](notes/55-serviceaccounts-and-tokens.md)  |
| `37c.png` | [55](notes/55-serviceaccounts-and-tokens.md)  |
| `37d.png` | [55](notes/55-serviceaccounts-and-tokens.md)  |
| `37e.png` | [55](notes/55-serviceaccounts-and-tokens.md)  |
| `37f.png` | [55](notes/55-serviceaccounts-and-tokens.md)  |
| `37g.png` | [55](notes/55-serviceaccounts-and-tokens.md)  |
| `37h.png` | [55](notes/55-serviceaccounts-and-tokens.md)  |

| File | Used in |
|---|---|
| `cr-crb.yaml` | [55](notes/55-serviceaccounts-and-tokens.md)  |
| `secret.yaml` | [55](notes/55-serviceaccounts-and-tokens.md)  |

### Day 38 · Admission Controllers in Kubernetes | Mutating & Validating | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 38 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ↳ Prerequisites: | [58](notes/58-admission-control.md)  |
| Introduction | [58](notes/58-admission-control.md)  |
| Request Lifecycle Recap | [58](notes/58-admission-control.md)  |
| Admission Controllers in Kubernetes | [58](notes/58-admission-control.md)  |
| ↳ Built-in Admission Controllers (ACs) | [58](notes/58-admission-control.md)  |
| ↳ Core Concepts: Mutating vs Validating Admission Controllers | [58](notes/58-admission-control.md)  |
| Built-in Admission Controller Plugins | [58](notes/58-admission-control.md)  |
| ↳ 1. Mutating Admission Controllers | [58](notes/58-admission-control.md)  |
| ↳ 2. Validating Admission Controllers | [58](notes/58-admission-control.md)  |
| ↳ 3. Dual-function Admission Controllers | [58](notes/58-admission-control.md)  |
| Custom Admission Controllers | [58](notes/58-admission-control.md)  |
| ↳ What is a Webhook? | [58](notes/58-admission-control.md)  |
| ↳ Classification of Admission Controllers (Summary) | [58](notes/58-admission-control.md)  |
| Real-World Use Cases & Tools | [58](notes/58-admission-control.md)  |
| ↳ 📘 Note: Default Admission Controllers in Kubernetes (Including kubeadm and KIND Clusters) | [58](notes/58-admission-control.md)  |
| ↳ 🔍 How Admission Controller Flags Work | [58](notes/58-admission-control.md)  |
| How to Enable Admission Controllers | [58](notes/58-admission-control.md)  |
| Custom Admission Controllers: Webhooks | [58](notes/58-admission-control.md)  |
| ↳ Architecture | [58](notes/58-admission-control.md)  |
| ↳ Sequence: | [58](notes/58-admission-control.md)  |
| Understanding OPA in Authorization vs Admission Control | [58](notes/58-admission-control.md)  |
| ↳ Why OPA for Authorization and Admission Control? | [58](notes/58-admission-control.md)  |
| ↳ Two Ways to Use OPA in Kubernetes | [58](notes/58-admission-control.md)  |
| ↳ 🔐 OPA as a Webhook Authorizer: When RBAC Isn’t Enough | [58](notes/58-admission-control.md)  |
| ↳ ✅ When to Use OPA for Webhook Authorization | [58](notes/58-admission-control.md)  |
| ↳ Common Webhook Authorization Policies with OPA | [58](notes/58-admission-control.md)  |
| ↳ Why Use This in Production If Admission Controllers Can Validate? | [58](notes/58-admission-control.md)  |
| ↳ OPA Gatekeeper as an Admission Controller (Validation + Mutation Layer) | [58](notes/58-admission-control.md)  |
| ↳ What Gatekeeper Sees (High Visibility Into Requests) | [58](notes/58-admission-control.md)  |
| ↳ What Gatekeeper Does (Validation and Mutation) | [58](notes/58-admission-control.md)  |
| ↳ Kyverno vs Gatekeeper (Why Kyverno Is Often Better for Many Teams) | [58](notes/58-admission-control.md)  |
| ↳ Examples of Admission Policies Gatekeeper Can Enforce | [58](notes/58-admission-control.md)  |
| ↳ Limitations of OPA Gatekeeper | [58](notes/58-admission-control.md)  |
| ↳ Best Suited For | [58](notes/58-admission-control.md)  |
| ↳ Clarifying the Confusion: Authorization vs Admission Validation | [58](notes/58-admission-control.md)  |
| ↳ What Do Enterprises Use in Production? | [58](notes/58-admission-control.md)  |
| ↳ Custom Admission Controllers Beyond OPA | [58](notes/58-admission-control.md)  |
| ↳ Decision-Making Framework: When to Use What? | [58](notes/58-admission-control.md)  |
| ↳ Admission Controller Execution Sequence | [58](notes/58-admission-control.md)  |
| ↳ Admission Controller Sequence | [58](notes/58-admission-control.md)  |
| ↳ Example: | [58](notes/58-admission-control.md)  |
| ↳ Key Takeaways | [58](notes/58-admission-control.md)  |
| ↳ Conclusion | [58](notes/58-admission-control.md)  |
| ↳ References | [58](notes/58-admission-control.md)  |

| Image | Used in |
|---|---|
| `38a.png` | [58](notes/58-admission-control.md)  |
| `38b.png` | [58](notes/58-admission-control.md)  |
| `38c.png` | [58](notes/58-admission-control.md)  |
| `38d.png` | [58](notes/58-admission-control.md)  |
| `38e.png` | [58](notes/58-admission-control.md)  |
| `38f.png` | [58](notes/58-admission-control.md)  |

### Day 39 · Custom Resources (CR) and Custom Resource Definitions (CRD) Explained with Demo | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 39 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ↳ Prerequisites: | [66](notes/66-crds-and-custom-controllers.md)  |
| ↳ Introduction | [66](notes/66-crds-and-custom-controllers.md)  |
| What is a Kubernetes Resource? | [66](notes/66-crds-and-custom-controllers.md)  |
| What is a Custom Resource? | [66](notes/66-crds-and-custom-controllers.md)  |
| Key Properties of Custom Resources | [66](notes/66-crds-and-custom-controllers.md)  |
| What is a CustomResourceDefinition (CRD)? | [66](notes/66-crds-and-custom-controllers.md)  |
| Do Built-in Resources Have Definitions Like CRDs? | [66](notes/66-crds-and-custom-controllers.md)  |
| ↳ Equivalent of CRD for Built-in Resources | [66](notes/66-crds-and-custom-controllers.md)  |
| ↳ Where Is This Resource Definition Stored? | [66](notes/66-crds-and-custom-controllers.md)  |
| Understanding Controllers and Custom Controllers in Kubernetes | [66](notes/66-crds-and-custom-controllers.md)  |
| Custom Controllers and the Control Loop Pattern | [66](notes/66-crds-and-custom-controllers.md)  |
| ↳ What Is a Controller's Job? | [66](notes/66-crds-and-custom-controllers.md)  |
| ↳ What Is a Control Loop? | [66](notes/66-crds-and-custom-controllers.md)  |
| ↳ Real-World Analogy: Thermostat | [66](notes/66-crds-and-custom-controllers.md)  |
| ↳ Example: Deployment Controller | [66](notes/66-crds-and-custom-controllers.md)  |
| ↳ Custom Controllers: Bringing Logic to CRDs | [66](notes/66-crds-and-custom-controllers.md)  |
| ↳ Tools for Writing Controllers | [66](notes/66-crds-and-custom-controllers.md)  |
| ↳ CKA Relevance | [66](notes/66-crds-and-custom-controllers.md)  |
| ↳ Not All Resources Have Controllers | [66](notes/66-crds-and-custom-controllers.md)  |
| Rule of Thumb | [66](notes/66-crds-and-custom-controllers.md)  |
| Demo: Creating a Realistic Custom Resource — BackupPolicy | [66](notes/66-crds-and-custom-controllers.md)  |
| ↳ Why Not Just Use a CronJob? | [66](notes/66-crds-and-custom-controllers.md)  |
| Step 1: Define the CRD — backup-policy-crd.yaml | [66](notes/66-crds-and-custom-controllers.md)  |
| Step 2: Apply the CRD | [66](notes/66-crds-and-custom-controllers.md)  |
| Step 3: Create a Custom Resource — mysql-backup.yaml | [66](notes/66-crds-and-custom-controllers.md)  |
| Step 4: Interact With the BackupPolicy Resource | [66](notes/66-crds-and-custom-controllers.md)  |
| Step 5: Clean Up | [66](notes/66-crds-and-custom-controllers.md)  |
| Recap: What’s Happening and Why It’s Useful | [66](notes/66-crds-and-custom-controllers.md)  |
| Role of a Controller in Managing Custom Resources like BackupPolicy | [66](notes/66-crds-and-custom-controllers.md)  |
| Real Production Scenario | [66](notes/66-crds-and-custom-controllers.md)  |
| ↳ Conclusion | [66](notes/66-crds-and-custom-controllers.md)  |
| ↳ References | [66](notes/66-crds-and-custom-controllers.md)  |

| Image | Used in |
|---|---|
| `39a.png` | [66](notes/66-crds-and-custom-controllers.md)  |
| `39b.png` | [66](notes/66-crds-and-custom-controllers.md)  |

| File | Used in |
|---|---|
| `cr.yaml` | [66](notes/66-crds-and-custom-controllers.md)  |
| `crd.yaml` | [66](notes/66-crds-and-custom-controllers.md)  |
| `deploy.yaml` | [66](notes/66-crds-and-custom-controllers.md)  |

### Day 40 · Kubernetes Operators Deep Dive with Hands-On Demo | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 40 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ↳ Prerequisites: | [67](notes/67-operators.md)  |
| ↳ Quick Recap | [67](notes/67-operators.md)  |
| Introduction | [67](notes/67-operators.md)  |
| Why Do We Need Kubernetes Operators? | [67](notes/67-operators.md)  |
| Understanding Stateless vs Stateful Applications in Kubernetes | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ What Is a Stateless Application? | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ What Is a Stateful Application? | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| Without Native Automation: Manual Responsibilities | [67](notes/67-operators.md)  |
| ↳ DevOps Team Responsibilities | [67](notes/67-operators.md)  |
| ↳ DBA Team Responsibilities | [67](notes/67-operators.md)  |
| What Is a Kubernetes Operator? | [67](notes/67-operators.md)  |
| ↳ Why It Matters | [67](notes/67-operators.md)  |
| Core Components of a Kubernetes Operator | [67](notes/67-operators.md)  |
| ↳ 📐 Operator Hierarchy | [67](notes/67-operators.md)  |
| ↳ Component Breakdown | [67](notes/67-operators.md)  |
| Don't Limit Your Imagination with Operators | [67](notes/67-operators.md)  |
| Helm and Kustomize Are Still Very Relevant | [67](notes/67-operators.md)  |
| Revisited: Our Custom BackupPolicy Operator | [67](notes/67-operators.md)  |
| ↳ What Makes a Mature Operator? | [67](notes/67-operators.md)  |
| How Can We Create Kubernetes Operators? | [67](notes/67-operators.md)  |
| ↳ Popular Operator Development Frameworks | [67](notes/67-operators.md)  |
| ↳ How Do You Choose? | [67](notes/67-operators.md)  |
| Step 1: Install the Operator Lifecycle Manager (OLM) | [67](notes/67-operators.md)  |
| Step 2: Install the Kube-Green Operator | [67](notes/67-operators.md)  |
| Step 3: Verify the Operator Installation | [67](notes/67-operators.md)  |
| ↳ Verify the CRD is Registered | [67](notes/67-operators.md)  |
| Step 4: Deploy a Sample Application | [67](notes/67-operators.md)  |
| Step 5: Create a SleepInfo Custom Resource | [67](notes/67-operators.md)  |
| ↳ Explanation of Fields | [67](notes/67-operators.md)  |
| Step 6: Observe the Operator in Action | [67](notes/67-operators.md)  |
| Conclusion | [67](notes/67-operators.md)  |
| References | [67](notes/67-operators.md)  |

| Image | Used in |
|---|---|
| `40a.png` | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| `40b.png` | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| `40c.png` | [67](notes/67-operators.md)  |
| `40d.png` | [67](notes/67-operators.md)  |

| File | Used in |
|---|---|
| `00-kube-green.yaml` | [67](notes/67-operators.md)  |
| `01-app.yaml` | [67](notes/67-operators.md)  |
| `02-sleep.yaml` | [67](notes/67-operators.md)  |
| `test.yaml` | [67](notes/67-operators.md)  |

### Day 41 · Pod Security in Kubernetes – Mastering Security Context & Linux Capabilities | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 41 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Introduction | [59](notes/59-securitycontext-and-capabilities.md)  |
| ↳ Why Pod Security? | [59](notes/59-securitycontext-and-capabilities.md)  |
| Pod Security | [59](notes/59-securitycontext-and-capabilities.md)  |
| 1. Security Context | [59](notes/59-securitycontext-and-capabilities.md)  |
| Security Context in Kubernetes: Field-by-Field Overview | [59](notes/59-securitycontext-and-capabilities.md)  |
| ↳ Pod-Level Fields (spec.securityContext) | [59](notes/59-securitycontext-and-capabilities.md)  |
| ↳ Container-Level Fields (spec.containers[].securityContext) | [59](notes/59-securitycontext-and-capabilities.md)  |
| ↳ Scope of Security Context Fields: Pod vs. Container | [59](notes/59-securitycontext-and-capabilities.md)  |
| Demo: Security Context | [59](notes/59-securitycontext-and-capabilities.md)  |
| ↳ Step 1: Deploy a Pod with Security Contexts Defined | [59](notes/59-securitycontext-and-capabilities.md)  |
| ↳ Step 2: Verification | [59](notes/59-securitycontext-and-capabilities.md)  |
| ↳ Step 3: Test Read-Only Root Filesystem | [59](notes/59-securitycontext-and-capabilities.md)  |
| ↳ Step 4: Test Privilege Escalation | [59](notes/59-securitycontext-and-capabilities.md)  |
| ↳ Step 5: Understanding whoami | [59](notes/59-securitycontext-and-capabilities.md)  |
| Demo 2: Writable Volume Using fsGroup | [59](notes/59-securitycontext-and-capabilities.md)  |
| ↳ Goal | [59](notes/59-securitycontext-and-capabilities.md)  |
| ↳ Step 1: Pod Definition | [59](notes/59-securitycontext-and-capabilities.md)  |
| ↳ Step 2: Verification | [59](notes/59-securitycontext-and-capabilities.md)  |
| ↳ What This Demonstrates | [59](notes/59-securitycontext-and-capabilities.md)  |
| ↳ Takeaway | [59](notes/59-securitycontext-and-capabilities.md)  |
| 2. Linux Capabilities | [59](notes/59-securitycontext-and-capabilities.md)  |
| ↳ What Are Linux Capabilities? | [59](notes/59-securitycontext-and-capabilities.md)  |
| ↳ Common Linux Capabilities | [59](notes/59-securitycontext-and-capabilities.md)  |
| Demo: Linux Capabilities | [59](notes/59-securitycontext-and-capabilities.md)  |
| ↳ Step 1: Deploy a Pod With Select Capabilities | [59](notes/59-securitycontext-and-capabilities.md)  |
| ↳ Step 2: Exec Into the Pod | [59](notes/59-securitycontext-and-capabilities.md)  |
| ↳ Step 3: Verify Added Capabilities Work | [59](notes/59-securitycontext-and-capabilities.md)  |
| ↳ Step 4: Verify Dropped Capabilities Fail | [59](notes/59-securitycontext-and-capabilities.md)  |
| ↳ Key Takeaway | [59](notes/59-securitycontext-and-capabilities.md)  |
| Conclusion | [59](notes/59-securitycontext-and-capabilities.md)  |
| References | [59](notes/59-securitycontext-and-capabilities.md)  |

| File | Used in |
|---|---|
| `00-default-pod.yaml` | [59](notes/59-securitycontext-and-capabilities.md)  |
| `01-security-context copy.yaml` | [59](notes/59-securitycontext-and-capabilities.md)  |
| `02-security-context.yaml` | [59](notes/59-securitycontext-and-capabilities.md)  |
| `03-capability-pod.yaml` | [59](notes/59-securitycontext-and-capabilities.md)  |

### Day 42 · Kubernetes Kustomize Explained with Practical Demos | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 42 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Introduction | [64](notes/64-kustomize.md)  |
| Why Kustomize? | [64](notes/64-kustomize.md)  |
| ↳ The Problem | [64](notes/64-kustomize.md)  |
| ↳ What If You Want to: | [64](notes/64-kustomize.md)  |
| ↳ What We Want | [64](notes/64-kustomize.md)  |
| ↳ Real-World Perspective | [64](notes/64-kustomize.md)  |
| Enter Kustomize | [64](notes/64-kustomize.md)  |
| What is Kustomize? | [64](notes/64-kustomize.md)  |
| Example: NGINX Deployment Across Dev, Staging, and Prod | [64](notes/64-kustomize.md)  |
| ↳ What We Do with Kustomize | [64](notes/64-kustomize.md)  |
| Recommended Folder Structure (Using patches: Inline) | [64](notes/64-kustomize.md)  |
| How Kustomize Works (Simplified Flow Diagram) | [64](notes/64-kustomize.md)  |
| Helm vs Kustomize | [64](notes/64-kustomize.md)  |
| ↳ Kustomize vs Helm | [64](notes/64-kustomize.md)  |
| ↳ Why Introduce Kustomize First? | [64](notes/64-kustomize.md)  |
| Feature Comparison: Helm vs Kustomize | [64](notes/64-kustomize.md)  |
| Installing Kustomize | [64](notes/64-kustomize.md)  |
| Understanding kustomization.yaml | [64](notes/64-kustomize.md)  |
| ↳ In the base/ Directory | [64](notes/64-kustomize.md)  |
| ↳ In the overlays/ Directory | [64](notes/64-kustomize.md)  |
| ↳ Why This Separation Matters | [64](notes/64-kustomize.md)  |
| ↳ Is base/ and overlays/ Mandatory? | [64](notes/64-kustomize.md)  |
| Demo 1: Kustomize Basics with Name Suffix Transformer | [64](notes/64-kustomize.md)  |
| ↳ Step 1: Directory Setup | [64](notes/64-kustomize.md)  |
| ↳ Step 2: Create the Base Configuration | [64](notes/64-kustomize.md)  |
| ↳ Explanation | [64](notes/64-kustomize.md)  |
| ↳ Step 3: Create Overlays for Environments | [64](notes/64-kustomize.md)  |
| ↳ Step 4: View Generated Manifests | [64](notes/64-kustomize.md)  |
| ↳ Step 5: Namespace Preparation and Applying Overlays | [64](notes/64-kustomize.md)  |
| ↳ Summary of What We Learned | [64](notes/64-kustomize.md)  |
| ↳ What’s Next? | [64](notes/64-kustomize.md)  |
| Demo 2: Applying Additional Transformers in Kustomize | [64](notes/64-kustomize.md)  |
| ↳ Objective | [64](notes/64-kustomize.md)  |
| ↳ Step 1: Update kustomization.yaml in overlays/dev | [64](notes/64-kustomize.md)  |
| ↳ Step 2: Field-by-Field Explanation | [64](notes/64-kustomize.md)  |
| ↳ Step 3: Replicate for Stage and Prod | [64](notes/64-kustomize.md)  |
| ↳ Step 4: Verify the Output | [64](notes/64-kustomize.md)  |
| ↳ Step 5: Apply the Kustomized Manifest | [64](notes/64-kustomize.md)  |
| ↳ Notes & Best Practices | [64](notes/64-kustomize.md)  |
| Kustomize Commands Must Run in a Directory with kustomization.yaml | [64](notes/64-kustomize.md)  |
| ↳ Why This Matters | [64](notes/64-kustomize.md)  |
| ↳ Example 1: Running Kustomize from Inside a Directory | [64](notes/64-kustomize.md)  |
| ↳ Example 2: Referencing Another Directory | [64](notes/64-kustomize.md)  |
| ↳ What If kustomization.yaml Is Missing? | [64](notes/64-kustomize.md)  |
| ↳ Final Note | [64](notes/64-kustomize.md)  |
| ↳ Documentation | [64](notes/64-kustomize.md)  |
| What Are Transformers in Kustomize? | [64](notes/64-kustomize.md)  |
| ↳ Examples of Transformers | [64](notes/64-kustomize.md)  |
| When Transformers Fall Short | [64](notes/64-kustomize.md)  |
| ↳ What Transformers Cannot Do | [64](notes/64-kustomize.md)  |
| Enter Patching | [64](notes/64-kustomize.md)  |
| ↳ What Are Patches? | [64](notes/64-kustomize.md)  |
| Categories of Patches in Kustomize | [64](notes/64-kustomize.md)  |
| Patching in Kustomize: From Strategic Merge to Inline Flexibility | [64](notes/64-kustomize.md)  |
| 1. patches (Generic Form) — Preferred | [64](notes/64-kustomize.md)  |
| ↳ Why Use This Method | [64](notes/64-kustomize.md)  |
| ↳ JSON Patch Operations (RFC 6902) | [64](notes/64-kustomize.md)  |
| Demo 3: patches (Generic Form) | [64](notes/64-kustomize.md)  |
| ↳ svc.yaml in Base | [64](notes/64-kustomize.md)  |
| ↳ overlays/dev/kustomization.yaml | [64](notes/64-kustomize.md)  |
| ↳ What We're Doing | [64](notes/64-kustomize.md)  |
| ↳ Note on Replace vs Add | [64](notes/64-kustomize.md)  |
| 2. patchesJson6902 — File-Based Patching (Deprecated) | [64](notes/64-kustomize.md)  |
| ↳ Why You Might Still See This | [64](notes/64-kustomize.md)  |
| ↳ How It Works (Legacy Understanding) | [64](notes/64-kustomize.md)  |
| Demo 4: patchesJson6902 (Deprecated) | [64](notes/64-kustomize.md)  |
| ↳ Project Structure | [64](notes/64-kustomize.md)  |
| ↳ overlays/dev/kustomization.yaml | [64](notes/64-kustomize.md)  |
| ↳ overlays/dev/patch-container.yaml | [64](notes/64-kustomize.md)  |
| ↳ What We're Doing | [64](notes/64-kustomize.md)  |
| ↳ How to Migrate Using kustomize edit fix | [64](notes/64-kustomize.md)  |
| ↳ Real-World Consideration | [64](notes/64-kustomize.md)  |
| ↳ Why It Was Popular (Before Deprecation) | [64](notes/64-kustomize.md)  |
| 3. patchesStrategicMerge (Deprecated) | [64](notes/64-kustomize.md)  |
| ↳ Why Learn It | [64](notes/64-kustomize.md)  |
| Demo 5: patchesStrategicMerge (Deprecated) | [64](notes/64-kustomize.md)  |
| ↳ Background | [64](notes/64-kustomize.md)  |
| ↳ Project Structure | [64](notes/64-kustomize.md)  |
| ↳ overlays/dev/kustomization.yaml | [64](notes/64-kustomize.md)  |
| ↳ overlays/dev/patch-container.yaml | [64](notes/64-kustomize.md)  |
| ↳ What We're Doing | [64](notes/64-kustomize.md)  |
| ↳ Why This Is Deprecated | [64](notes/64-kustomize.md)  |
| ↳ How to Migrate Using kustomize edit fix | [64](notes/64-kustomize.md)  |
| Final Comparison | [64](notes/64-kustomize.md)  |
| ↳ Notes: | [64](notes/64-kustomize.md)  |
| Conclusion | [64](notes/64-kustomize.md)  |
| References | [64](notes/64-kustomize.md)  |

| Image | Used in |
|---|---|
| `42a.png` | [64](notes/64-kustomize.md)  |

| File | Used in |
|---|---|
| `demo1/base/deploy.yaml` | [64](notes/64-kustomize.md)  |
| `demo1/base/kustomization.yaml` | [64](notes/64-kustomize.md)  |
| `demo1/base/svc.yaml` | [64](notes/64-kustomize.md)  |
| `demo1/overlays/dev/kustomization.yaml` | [64](notes/64-kustomize.md)  |
| `demo1/overlays/prod/kustomization.yaml` | [64](notes/64-kustomize.md)  |
| `demo1/overlays/stage/kustomization.yaml` | [64](notes/64-kustomize.md)  |
| `demo2/base/deploy.yaml` | [64](notes/64-kustomize.md)  |
| `demo2/base/kustomization.yaml` | [64](notes/64-kustomize.md)  |
| `demo2/base/svc.yaml` | [64](notes/64-kustomize.md)  |
| `demo2/overlays/dev/kustomization.yaml` | [64](notes/64-kustomize.md)  |
| `demo2/overlays/prod/kustomization.yaml` | [64](notes/64-kustomize.md)  |
| `demo2/overlays/stage/kustomization.yaml` | [64](notes/64-kustomize.md)  |
| `demo3_patches/base/deploy.yaml` | [64](notes/64-kustomize.md)  |
| `demo3_patches/base/kustomization.yaml` | [64](notes/64-kustomize.md)  |
| `demo3_patches/base/svc.yaml` | [64](notes/64-kustomize.md)  |
| `demo3_patches/overlays/dev/kustomization.yaml` | [64](notes/64-kustomize.md)  |
| `demo4_patchesjson/base/deploy.yaml` | [64](notes/64-kustomize.md)  |
| `demo4_patchesjson/base/kustomization.yaml` | [64](notes/64-kustomize.md)  |
| `demo4_patchesjson/base/svc.yaml` | [64](notes/64-kustomize.md)  |
| `demo4_patchesjson/overlays/dev/kustomization.yaml` | [64](notes/64-kustomize.md)  |
| `demo4_patchesjson/overlays/dev/patch-container.yaml` | [64](notes/64-kustomize.md)  |
| `demo5_patchesmerge/base/deploy.yaml` | [64](notes/64-kustomize.md)  |
| `demo5_patchesmerge/base/kustomization.yaml` | [64](notes/64-kustomize.md)  |
| `demo5_patchesmerge/base/svc.yaml` | [64](notes/64-kustomize.md)  |
| `demo5_patchesmerge/overlays/dev/kustomization.yaml` | [64](notes/64-kustomize.md)  |
| `demo5_patchesmerge/overlays/dev/patch-container.yaml` | [64](notes/64-kustomize.md)  |

### Day 43 · Helm Charts for Beginners | Helm in Kubernetes with 3 Practical Demos | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 43 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Introduction | [65](notes/65-helm.md)  |
| Why Helm? | [65](notes/65-helm.md)  |
| ↳ The Problem | [65](notes/65-helm.md)  |
| ↳ What Makes Helm Special (Compared to Kustomize)? | [65](notes/65-helm.md)  |
| ↳ Real-World Perspective | [65](notes/65-helm.md)  |
| ↳ Enter Helm | [65](notes/65-helm.md)  |
| What is Helm? | [65](notes/65-helm.md)  |
| ↳ Why Helm Needs to Understand Your Application | [65](notes/65-helm.md)  |
| Installing Helm | [65](notes/65-helm.md)  |
| Helm Core Concepts | [65](notes/65-helm.md)  |
| ↳ 1. Chart | [65](notes/65-helm.md)  |
| ↳ 2. Repository | [65](notes/65-helm.md)  |
| ↳ Enter Artifact Hub | [65](notes/65-helm.md)  |
| ↳ 3. Release | [65](notes/65-helm.md)  |
| ↳ Helm Tracks Release Revisions | [65](notes/65-helm.md)  |
| ↳ Helm vs Docker Analogy | [65](notes/65-helm.md)  |
| ↳ Putting It Together | [65](notes/65-helm.md)  |
| ↳ Example | [65](notes/65-helm.md)  |
| Basic Helm Commands | [65](notes/65-helm.md)  |
| ↳ Add a Repository | [65](notes/65-helm.md)  |
| ↳ List Added Repositories | [65](notes/65-helm.md)  |
| ↳ Search for a Chart | [65](notes/65-helm.md)  |
| ↳ Search Artifact Hub for Charts | [65](notes/65-helm.md)  |
| ↳ Filter Artifact Hub Results for Bitnami | [65](notes/65-helm.md)  |
| ↳ Install a Chart | [65](notes/65-helm.md)  |
| ↳ Uninstall a Chart | [65](notes/65-helm.md)  |
| ↳ Upgrade a Chart | [65](notes/65-helm.md)  |
| Creating a Helm Chart | [65](notes/65-helm.md)  |
| ↳ What These Files Mean | [65](notes/65-helm.md)  |
| Demo 1: Writing Your Own Helm Chart from Scratch | [65](notes/65-helm.md)  |
| ↳ Objective | [65](notes/65-helm.md)  |
| Step 1: Scaffold Helm Directory | [65](notes/65-helm.md)  |
| Step 2: Add Basic (Plain) YAMLs | [65](notes/65-helm.md)  |
| ↳ deploy.yaml (Plain) | [65](notes/65-helm.md)  |
| ↳ svc.yaml (Plain) | [65](notes/65-helm.md)  |
| ↳ Substep: This Chart Is Already Installable — But Not Reusable | [65](notes/65-helm.md)  |
| ↳ Lesson | [65](notes/65-helm.md)  |
| Step 3: Introduce Helm Templating | [65](notes/65-helm.md)  |
| ↳ Updated deploy.yaml (Templatized) | [65](notes/65-helm.md)  |
| ↳ Updated svc.yaml (Templatized) | [65](notes/65-helm.md)  |
| Step 4: Update values.yaml | [65](notes/65-helm.md)  |
| Step 5: Update Chart.yaml | [65](notes/65-helm.md)  |
| Step 6: Understanding Helm Templating | [65](notes/65-helm.md)  |
| ↳ .Values – User-defined inputs (from values.yaml) | [65](notes/65-helm.md)  |
| ↳ .Release – Information about the release | [65](notes/65-helm.md)  |
| ↳ .Chart – Metadata from Chart.yaml | [65](notes/65-helm.md)  |
| ↳ .Capabilities – Info about Kubernetes cluster | [65](notes/65-helm.md)  |
| Step 7: Helm Verification Commands | [65](notes/65-helm.md)  |
| ↳ Lint the Chart | [65](notes/65-helm.md)  |
| ↳ Render Templates | [65](notes/65-helm.md)  |
| ↳ Run with Dry-Run | [65](notes/65-helm.md)  |
| Step 8: Install the Chart | [65](notes/65-helm.md)  |
| ↳ Helm Verification | [65](notes/65-helm.md)  |
| ↳ Kubernetes Verification | [65](notes/65-helm.md)  |
| Optional: Uninstall the Chart | [65](notes/65-helm.md)  |
| Final Notes | [65](notes/65-helm.md)  |
| ↳ Why Helm Templating Matters | [65](notes/65-helm.md)  |
| ↳ Multiple Releases from the Same Chart | [65](notes/65-helm.md)  |
| Demo 2: Helm Upgrade and Rollback in Action | [65](notes/65-helm.md)  |
| ↳ Our Task | [65](notes/65-helm.md)  |
| ↳ Step 1: Verify Current Release and Revision | [65](notes/65-helm.md)  |
| ↳ Step 2: Upgrade the Chart and Image Version | [65](notes/65-helm.md)  |
| ↳ Step 3: Perform Helm Upgrade | [65](notes/65-helm.md)  |
| ↳ Step 4: Post-Upgrade Validation | [65](notes/65-helm.md)  |
| ↳ Step 5: Rollback to Previous Version | [65](notes/65-helm.md)  |
| ↳ Step 6: Post-Rollback Verification | [65](notes/65-helm.md)  |
| ↳ Bonus: Try Helm Upgrade with --reuse-values | [65](notes/65-helm.md)  |
| ↳ Key Commands | [65](notes/65-helm.md)  |
| Demo 3: Multi-Environment Deployments with Helm | [65](notes/65-helm.md)  |
| ↳ Objective | [65](notes/65-helm.md)  |
| ↳ Step 0: Clean Up the Environment | [65](notes/65-helm.md)  |
| ↳ Step 1: Create Environment Namespaces | [65](notes/65-helm.md)  |
| ↳ Step 2: Ensure Chart Structure Is Production-Ready | [65](notes/65-helm.md)  |
| ↳ Step 3: Update Chart.yaml | [65](notes/65-helm.md)  |
| ↳ What appVersion Should Represent | [65](notes/65-helm.md)  |
| ↳ Step 4: Create Environment-Specific values.yaml Files | [65](notes/65-helm.md)  |
| ↳ Step 5: Verify and Deploy to Each Environment | [65](notes/65-helm.md)  |
| ↳ (A) Verify Before You Deploy | [65](notes/65-helm.md)  |
| ↳ (B) Deploy to Each Environment | [65](notes/65-helm.md)  |
| ↳ Important Note: Helm’s Tracking vs Kubernetes Deployment | [65](notes/65-helm.md)  |
| ↳ Best Practice (Production Style) | [65](notes/65-helm.md)  |
| ↳ Step 6: Verify the Deployments | [65](notes/65-helm.md)  |
| ↳ Step 7: Upgrade Dev Environment Only | [65](notes/65-helm.md)  |
| ↳ Summary | [65](notes/65-helm.md)  |
| ↳ Why Namespaces Matter | [65](notes/65-helm.md)  |
| Conclusion | [65](notes/65-helm.md)  |
| References | [65](notes/65-helm.md)  |

| Image | Used in |
|---|---|
| `43a.png` | [65](notes/65-helm.md)  |
| `43b.png` | [65](notes/65-helm.md)  |

| File | Used in |
|---|---|
| `demo1/app1-chart/.helmignore` | [65](notes/65-helm.md)  |
| `demo1/app1-chart/Chart.yaml` | [65](notes/65-helm.md)  |
| `demo1/app1-chart/templates/deploy.yaml` | [65](notes/65-helm.md)  |
| `demo1/app1-chart/templates/svc.yaml` | [65](notes/65-helm.md)  |
| `demo1/app1-chart/values.yaml` | [65](notes/65-helm.md)  |
| `demo2/app1-chart/.helmignore` | [65](notes/65-helm.md)  |
| `demo2/app1-chart/Chart.yaml` | [65](notes/65-helm.md)  |
| `demo2/app1-chart/templates/deploy.yaml` | [65](notes/65-helm.md)  |
| `demo2/app1-chart/templates/svc.yaml` | [65](notes/65-helm.md)  |
| `demo2/app1-chart/values.yaml` | [65](notes/65-helm.md)  |
| `demo3/app1-chart/.helmignore` | [65](notes/65-helm.md)  |
| `demo3/app1-chart/Chart.yaml` | [65](notes/65-helm.md)  |
| `demo3/app1-chart/templates/deploy.yaml` | [65](notes/65-helm.md)  |
| `demo3/app1-chart/templates/svc.yaml` | [65](notes/65-helm.md)  |
| `demo3/app1-chart/values-dev.yaml` | [65](notes/65-helm.md)  |
| `demo3/app1-chart/values-prod.yaml` | [65](notes/65-helm.md)  |
| `demo3/app1-chart/values-stage.yaml` | [65](notes/65-helm.md)  |
| `my-chart/.helmignore` | [65](notes/65-helm.md)  |
| `my-chart/Chart.yaml` | [65](notes/65-helm.md)  |
| `my-chart/templates/_helpers.tpl` | [65](notes/65-helm.md)  |
| `my-chart/templates/deployment.yaml` | [65](notes/65-helm.md)  |
| `my-chart/templates/hpa.yaml` | [65](notes/65-helm.md)  |
| `my-chart/templates/ingress.yaml` | [65](notes/65-helm.md)  |
| `my-chart/templates/NOTES.txt` | [65](notes/65-helm.md)  |
| `my-chart/templates/service.yaml` | [65](notes/65-helm.md)  |
| `my-chart/templates/serviceaccount.yaml` | [65](notes/65-helm.md)  |
| `my-chart/templates/tests/test-connection.yaml` | [65](notes/65-helm.md)  |
| `my-chart/values.yaml` | [65](notes/65-helm.md)  |

### Day 44 · MASTER StatefulSets in Kubernetes | Real-World Multi-AZ Amazon EKS Demo | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 44 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Introduction | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| Understanding Stateless vs Stateful Applications | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| The Classic Three-Tier Architecture | [40](notes/40-statefulsets.md)  |
| A Common Problem: When the Frontend Stores Session Info Locally | [40](notes/40-statefulsets.md)  |
| Backend Tier: A Classic Stateless Component | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| Database Tier: Always Stateful | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| But Apps Want to Be Stateless — Why? | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| Upgrading the Architecture: Using Amazon ElastiCache for Session Management | [40](notes/40-statefulsets.md)  |
| The Takeaway | [40](notes/40-statefulsets.md)  |
| What It Takes to Run a Database | [40](notes/40-statefulsets.md)  |
| ↳ Typical Production Setup | [40](notes/40-statefulsets.md)  |
| Replication Mechanics | [40](notes/40-statefulsets.md)  |
| ↳ 1. Initial Clone | [40](notes/40-statefulsets.md)  |
| ↳ 2. Continuous Replication | [40](notes/40-statefulsets.md)  |
| Problem: What Happens If a Replica Crashes? | [40](notes/40-statefulsets.md)  |
| ↳ Why? | [40](notes/40-statefulsets.md)  |
| Why This Matters in Kubernetes | [40](notes/40-statefulsets.md)  |
| Why StatefulSets? | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ 1. No Predictable Pod Naming | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ 2. No Stable DNS Records Per Pod | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ 3. No Ordered Pod Deployment or Termination | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ 4. No Pod-Scoped Persistent Volumes | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| So, What Solves These Problems? | [40](notes/40-statefulsets.md)  |
| What are StatefulSets? | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ 1. Stable and Predictable Pod Naming | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ 2. Stable Network Identity (DNS) | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ 🔁 How They're Different but Complementary | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ 3. Ordered Deployment and Termination | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ 4. Persistent Volume Reattachment | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ StatefulSet Key Characteristics | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| Comparison with Deployments | [40](notes/40-statefulsets.md)  |
| Persistent Storage in StatefulSets | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ Why Block Storage? | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ Real-World Example: Block Storage with RWOP | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ Why You Must Understand Access Modes | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ Storage Behavior in StatefulSets | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ Why Shared Storage Backend Matters | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| Recap: Why StatefulSets Are Critical for Databases | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| Demo 1: StatefulSet Implementation in a Multi-AZ Amazon EKS Cluster | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| Cluster Details | [40](notes/40-statefulsets.md)  |
| What This Demo Proves | [40](notes/40-statefulsets.md)  |
| Prerequisites | [40](notes/40-statefulsets.md)  |
| ↳ 1. Understand Storage Concepts | [40](notes/40-statefulsets.md)  |
| ↳ 2. Install AWS CLI and eksctl | [40](notes/40-statefulsets.md)  |
| ↳ 3. Provision the EKS Cluster | [40](notes/40-statefulsets.md)  |
| ↳ 4. Install the Amazon EBS CSI Driver | [40](notes/40-statefulsets.md)  |
| Step 1: Create a StorageClass | [40](notes/40-statefulsets.md)  |
| Step 2: Create Namespace | [40](notes/40-statefulsets.md)  |
| Step 3: Create a Headless Service | [40](notes/40-statefulsets.md)  |
| ↳ Why a Headless Service? | [40](notes/40-statefulsets.md)  |
| ↳ Why Not Use a ClusterIP Service? | [40](notes/40-statefulsets.md)  |
| ↳ Why Not Hardcode Pod Names in Your App? | [40](notes/40-statefulsets.md)  |
| Step 4: Deploy the MySQL StatefulSet | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| Step 5: Switch to the mysql-ha Namespace | [40](notes/40-statefulsets.md)  |
| Step 6: Observe StatefulSet Behavior | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ Single AZ Scenario: | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| Step 7: Simulate Failures | [40](notes/40-statefulsets.md)  |
| ↳ Important Notes on StatefulSet PVC Lifecycle | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| ↳ If You Truly Want to Clean Everything | [40](notes/40-statefulsets.md)  |
| ↳ Key Pointers | [40](notes/40-statefulsets.md)  |
| Demo 2: Chaos Engineering: Verify PVC Reattachment and Data Persistence | [40](notes/40-statefulsets.md)  |
| ↳ Step 1: Identify the AZ with 2 Nodes | [40](notes/40-statefulsets.md)  |
| ↳ Step 2: Find the Pod Running in That AZ | [40](notes/40-statefulsets.md)  |
| ↳ Step 3: Write a File to the PVC | [40](notes/40-statefulsets.md)  |
| ↳ Step 4: Terminate the Node Hosting the Pod | [40](notes/40-statefulsets.md)  |
| ↳ Step 5: Confirm PVC Reattachment and Data Recovery | [40](notes/40-statefulsets.md)  |
| ↳ Outcome | [40](notes/40-statefulsets.md)  |
| Division of Responsibilities: DevOps Engineer vs. DBA in Kubernetes-Based Database Deployments | [40](notes/40-statefulsets.md)  |
| ↳ DevOps Engineer Responsibilities | [40](notes/40-statefulsets.md)  |
| ↳ DBA Responsibilities | [40](notes/40-statefulsets.md)  |
| ↳ Optional but Recommended: Use Kubernetes Operators | [40](notes/40-statefulsets.md)  |
| Conclusion | [40](notes/40-statefulsets.md)  |
| References | [40](notes/40-statefulsets.md)  |

| Image | Used in |
|---|---|
| `44a.png` | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| `44b.png` | [05](notes/05-app-architecture-monolith-3tier-microservices.md)  |
| `44c.png` | [40](notes/40-statefulsets.md)  |
| `44d.png` | [40](notes/40-statefulsets.md)  |
| `44e.png` | [40](notes/40-statefulsets.md)  |
| `44f.png` | [40](notes/40-statefulsets.md)  |

| File | Used in |
|---|---|
| `01-sc.yaml` | [40](notes/40-statefulsets.md)  |
| `02-mysql-hs.yaml` | [40](notes/40-statefulsets.md)  |
| `03-mysql-sts.yaml` | [40](notes/40-statefulsets.md)  |
| `eks-config.yaml` | [40](notes/40-statefulsets.md)  |

### Day 45 · Pull Private Images in Kubernetes with imagePullSecrets & ServiceAccount | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 45 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Introduction | [62](notes/62-private-registries.md)  |
| 1. Nomenclature: Image Registries, Repositories, and Images | [20](notes/20-images-and-pull-policy.md)  |
| ↳ Image Registry | [20](notes/20-images-and-pull-policy.md)  |
| ↳ Repository | [20](notes/20-images-and-pull-policy.md)  |
| ↳ Repository Identifier Format | [20](notes/20-images-and-pull-policy.md)  |
| ↳ Image | [20](notes/20-images-and-pull-policy.md)  |
| ↳ Image Identifier Syntax | [20](notes/20-images-and-pull-policy.md)  |
| ↳ Defaulting Behavior in Kubernetes | [20](notes/20-images-and-pull-policy.md)  |
| ↳ Explicit Examples | [20](notes/20-images-and-pull-policy.md)  |
| 2. Public vs Private Registries | [62](notes/62-private-registries.md)  |
| 3. The Problem: ImagePullBackOff | [62](notes/62-private-registries.md)  |
| ↳ 4. How Kubernetes Authenticates to Private Image Registries | [62](notes/62-private-registries.md)  |
| 5. Step-by-Step Demos | [62](notes/62-private-registries.md)  |
| Demo 0: Build and Push a Private Image to Docker Hub | [62](notes/62-private-registries.md)  |
| ↳ Step 1: Application Code – app.py | [62](notes/62-private-registries.md)  |
| ↳ Step 2: Create a Dockerfile | [62](notes/62-private-registries.md)  |
| ↳ Step 3: Build and Push the Docker Image | [62](notes/62-private-registries.md)  |
| ↳ Step 4: Make the Repository Private | [62](notes/62-private-registries.md)  |
| Demo 1: Authenticate to a Private Registry Using imagePullSecrets | [62](notes/62-private-registries.md)  |
| ↳ Step 1: Create a Docker Registry Secret | [62](notes/62-private-registries.md)  |
| ↳ Step 2: Use the Secret in a Pod Manifest | [62](notes/62-private-registries.md)  |
| ↳ Step 3: Decode the Secret (Handle with Caution) | [62](notes/62-private-registries.md)  |
| Demo 2: Configure Default ServiceAccount with Pull Secret (Namespace-Wide) | [62](notes/62-private-registries.md)  |
| ↳ Step 1: Create Namespace and Confirm Default ServiceAccount | [62](notes/62-private-registries.md)  |
| ↳ Step 2: Create the Pull Secret in the New Namespace | [62](notes/62-private-registries.md)  |
| ↳ Step 3: Patch the Default ServiceAccount (Option 1: CLI) | [62](notes/62-private-registries.md)  |
| ↳ Step 4: (Alternative) Manual Edit of ServiceAccount | [62](notes/62-private-registries.md)  |
| ↳ Step 5: Verify the Patch | [62](notes/62-private-registries.md)  |
| ↳ Step 6: Deploy a Pod Without Explicit imagePullSecrets | [62](notes/62-private-registries.md)  |
| ↳ Step 7: Confirm Pod Is Running | [62](notes/62-private-registries.md)  |
| Demo 3: Use a Custom ServiceAccount with Pull Secrets | [62](notes/62-private-registries.md)  |
| ↳ Step 1: Ensure the Secret Exists in the Namespace | [62](notes/62-private-registries.md)  |
| ↳ Step 2: Create a Custom ServiceAccount | [62](notes/62-private-registries.md)  |
| ↳ Step 3: Attach the Image Pull Secret to the Custom SA | [62](notes/62-private-registries.md)  |
| ↳ Step 4: Verify the Patch | [62](notes/62-private-registries.md)  |
| ↳ Step 5: Create a Pod That Uses the Custom ServiceAccount | [62](notes/62-private-registries.md)  |
| ↳ Step 6: Confirm Pod Status | [62](notes/62-private-registries.md)  |
| ↳ Key Takeaways | [62](notes/62-private-registries.md)  |
| Conclusion | [62](notes/62-private-registries.md)  |
| References | [62](notes/62-private-registries.md)  |

| Image | Used in |
|---|---|
| `45a.png` | [20](notes/20-images-and-pull-policy.md)  |
| `45b.png` | [62](notes/62-private-registries.md)  |

| File | Used in |
|---|---|
| `01-pod.yaml` | [62](notes/62-private-registries.md)  |
| `02-pod.yaml` | [62](notes/62-private-registries.md)  |
| `03-pod.yaml` | [62](notes/62-private-registries.md)  |
| `app1/app.py` | [62](notes/62-private-registries.md)  |
| `app1/Dockerfile` | [62](notes/62-private-registries.md)  |

### Day 46 · Pod Priority, PriorityClass, and Preemption in Kubernetes | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 46 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Introduction | [47](notes/47-priority-preemption-eviction.md)  |
| Why Pod Priority and Preemption Exist | [47](notes/47-priority-preemption-eviction.md)  |
| What Is a PriorityClass and How to Use It | [47](notes/47-priority-preemption-eviction.md)  |
| Understanding Preemption and Scheduler Behavior | [47](notes/47-priority-preemption-eviction.md)  |
| Special Notes and Advanced Insights | [47](notes/47-priority-preemption-eviction.md) [41](notes/41-requests-limits-qos-quotas.md)  |
| Demo: Pod Priority and Preemption in Action | [47](notes/47-priority-preemption-eviction.md)  |
| ↳ Step 1: Inspect Your Cluster’s Capacity and Existing PriorityClasses | [47](notes/47-priority-preemption-eviction.md)  |
| ↳ Step 2: Create PriorityClasses | [47](notes/47-priority-preemption-eviction.md)  |
| ↳ Optional Field: preemptionPolicy: Never | [47](notes/47-priority-preemption-eviction.md)  |
| ↳ Step 3: Deploy Low-Priority Pods | [47](notes/47-priority-preemption-eviction.md)  |
| ↳ Step 4: Deploy High-Priority Pods and Observe Preemption | [47](notes/47-priority-preemption-eviction.md)  |
| ↳ Key Takeways: | [47](notes/47-priority-preemption-eviction.md)  |
| Conclusion | [47](notes/47-priority-preemption-eviction.md)  |
| References | [47](notes/47-priority-preemption-eviction.md)  |

| File | Used in |
|---|---|
| `01-priorityclass.yaml` | [47](notes/47-priority-preemption-eviction.md)  |
| `02-low-priority-nginx.yaml` | [47](notes/47-priority-preemption-eviction.md)  |
| `03-high-priority-nginx.yaml` | [47](notes/47-priority-preemption-eviction.md)  |

### Day 47 · Kubernetes Network Policies Explained with Real-World Demo | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 47 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Introduction | [61](notes/61-network-policies.md)  |
| Understanding the Context: Our 3-Tier Application | [61](notes/61-network-policies.md)  |
| How This Application Works | [61](notes/61-network-policies.md)  |
| Why Network Policies? | [61](notes/61-network-policies.md)  |
| What are Network Policies | [61](notes/61-network-policies.md)  |
| Who Enforces Network Policies? | [61](notes/61-network-policies.md)  |
| Ingress vs Egress – What’s the Difference? | [61](notes/61-network-policies.md)  |
| ↳ Example Flow | [61](notes/61-network-policies.md)  |
| Applying the Principle of Least Privilege | [61](notes/61-network-policies.md)  |
| ↳ Frontend (Web Tier) | [61](notes/61-network-policies.md)  |
| ↳ Backend (App Tier) | [61](notes/61-network-policies.md)  |
| ↳ Database (DB Tier) | [61](notes/61-network-policies.md)  |
| How Network Policies Work in Kubernetes | [61](notes/61-network-policies.md)  |
| ↳ Default Behavior | [61](notes/61-network-policies.md)  |
| ↳ Stateful Nature of Network Policies | [61](notes/61-network-policies.md)  |
| Types of NetworkPolicies | [61](notes/61-network-policies.md)  |
| ↳ 1. Pod-Based (podSelector) | [61](notes/61-network-policies.md)  |
| ↳ 2. Namespace-Based (namespaceSelector) | [61](notes/61-network-policies.md)  |
| ↳ 3. IP-Based (ipBlock) | [61](notes/61-network-policies.md)  |
| Demo: Enforcing Least Privilege Using NetworkPolicies | [61](notes/61-network-policies.md)  |
| ↳ Demo Tasks | [61](notes/61-network-policies.md)  |
| ↳ Step 1: Preparing the Cluster (KIND + Calico) | [61](notes/61-network-policies.md)  |
| ↳ Delete Existing Cluster (Optional) | [61](notes/61-network-policies.md)  |
| ↳ Create KIND Config with Default CNI Disabled | [61](notes/61-network-policies.md)  |
| ↳ Check Node Status | [61](notes/61-network-policies.md)  |
| ↳ Install Calico (CNI with NetworkPolicy Support) | [61](notes/61-network-policies.md)  |
| ↳ Verify Calico Installation | [61](notes/61-network-policies.md)  |
| Step 2: Creating Namespace and Deploying Frontend, Backend, and DB | [61](notes/61-network-policies.md)  |
| ↳ Creating the Namespace | [61](notes/61-network-policies.md)  |
| ↳ Deploying Frontend Resources | [61](notes/61-network-policies.md)  |
| ↳ Deploying Backend Resources | [61](notes/61-network-policies.md)  |
| ↳ Deploying Database Resources | [61](notes/61-network-policies.md)  |
| ↳ Step 3: Verify the Current State — Everyone Can Talk to Everyone | [61](notes/61-network-policies.md)  |
| ↳ Validate Communication from Frontend Pod | [61](notes/61-network-policies.md)  |
| ↳ Validate Communication from Backend Pod | [61](notes/61-network-policies.md)  |
| ↳ Optional: Validate from DB Pod (StatefulSet) | [61](notes/61-network-policies.md)  |
| Step 4: Creation of NetworkPolicies for All 3 Tiers | [61](notes/61-network-policies.md)  |
| ↳ 04-db-netpol.yaml – Allow Only Backend Pods to Reach DB on Port 3306 | [61](notes/61-network-policies.md)  |
| ↳ 05-revised-backend-netpol.yaml – Allow Ingress from Frontend and Egress to DB & DNS | [61](notes/61-network-policies.md)  |
| ↳ Explanation | [61](notes/61-network-policies.md)  |
| ↳ Why Do We Need to Allow DNS in Egress? | [61](notes/61-network-policies.md)  |
| ↳ Notes | [61](notes/61-network-policies.md)  |
| ↳ Apply and Verify the Policy | [61](notes/61-network-policies.md)  |
| ↳ Example 1: Combined in a single to: block (AND logic) | [61](notes/61-network-policies.md)  |
| ↳ Example 2: Written as two separate to: entries (OR logic) | [61](notes/61-network-policies.md)  |
| ↳ Table | [61](notes/61-network-policies.md)  |
| ↳ 06-frontend-netpol.yaml – Allow Ingress from External Clients & Egress to Backend and DNS | [61](notes/61-network-policies.md)  |
| ↳ Explanation | [61](notes/61-network-policies.md)  |
| ↳ Apply and Verify | [61](notes/61-network-policies.md)  |
| ↳ Key Pointers | [61](notes/61-network-policies.md)  |
| ↳ Step 5 – Verification of Connectivity | [61](notes/61-network-policies.md)  |
| Task for You | [61](notes/61-network-policies.md)  |
| AND vs OR: How from: Rules Are Evaluated | [61](notes/61-network-policies.md)  |
| ↳ What This Actually Means | [61](notes/61-network-policies.md)  |
| ↳ Want Both Conditions to Apply (AND)? | [61](notes/61-network-policies.md)  |
| ↳ Realistic Example Scenario | [61](notes/61-network-policies.md)  |
| Conclusion | [61](notes/61-network-policies.md)  |
| References | [61](notes/61-network-policies.md)  |

| Image | Used in |
|---|---|
| `47a.png` | [61](notes/61-network-policies.md)  |
| `47b.png` | [61](notes/61-network-policies.md)  |
| `47c.png` | [61](notes/61-network-policies.md)  |

| File | Used in |
|---|---|
| `calico.yaml` | [61](notes/61-network-policies.md)  |
| `demo/00-namespace.yaml` | [61](notes/61-network-policies.md)  |
| `demo/01-frontend.yaml` | [61](notes/61-network-policies.md) [85](notes/85-production-readiness-checklist.md)  |
| `demo/02-backend.yaml` | [61](notes/61-network-policies.md)  |
| `demo/03-db.yaml` | [61](notes/61-network-policies.md)  |
| `demo/04-db-netpol.yaml` | [61](notes/61-network-policies.md)  |
| `demo/05-backend-netpol.yaml` | [61](notes/61-network-policies.md) [85](notes/85-production-readiness-checklist.md)  |
| `demo/05-revised-backend-netpol.yaml` | [61](notes/61-network-policies.md)  |
| `demo/06-frontend-netpol.yaml` | [61](notes/61-network-policies.md)  |
| `kind-cluster.yaml` | [61](notes/61-network-policies.md)  |

### Day 48 · Kubernetes DNS Explained | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 48 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ↳ Prerequisites for This Lecture | [33](notes/33-cluster-dns-and-coredns.md)  |
| Introduction | [33](notes/33-cluster-dns-and-coredns.md)  |
| CoreDNS in Kubernetes | [33](notes/33-cluster-dns-and-coredns.md)  |
| Understanding DNS Mapping in Kubernetes with CoreDNS | [33](notes/33-cluster-dns-and-coredns.md)  |
| FQDNs in Kubernetes: Services, Pods, and StatefulSets | [33](notes/33-cluster-dns-and-coredns.md)  |
| ↳ FQDN Syntax Patterns | [33](notes/33-cluster-dns-and-coredns.md)  |
| ↳ Kubernetes DNS Breakdown: A Visual Guide | [33](notes/33-cluster-dns-and-coredns.md)  |
| ↳ How CoreDNS Manages and Resolves Names | [33](notes/33-cluster-dns-and-coredns.md)  |
| ↳ Explanation by Type | [33](notes/33-cluster-dns-and-coredns.md)  |
| ↳ Kubernetes DNS: What Gets Created and How It Resolves | [33](notes/33-cluster-dns-and-coredns.md)  |
| ↳ Notes: | [33](notes/33-cluster-dns-and-coredns.md)  |
| DNS Configuration Inside a Pod | [33](notes/33-cluster-dns-and-coredns.md)  |
| ↳ Understanding /etc/resolv.conf and DNS Search Order in Kubernetes | [33](notes/33-cluster-dns-and-coredns.md)  |
| /etc/hosts vs /etc/resolv.conf | [33](notes/33-cluster-dns-and-coredns.md)  |
| Verifying DNS Records in Kubernetes | [33](notes/33-cluster-dns-and-coredns.md)  |
| ↳ Step 1: Create a Test Service in the Default Namespace | [33](notes/33-cluster-dns-and-coredns.md)  |
| ↳ Step 2: Cross-Namespace DNS Resolution | [33](notes/33-cluster-dns-and-coredns.md)  |
| ↳ ⚠️ Why nslookup app1-svc Would Fail in This Case | [33](notes/33-cluster-dns-and-coredns.md)  |
| ↳ Key Takeaway | [33](notes/33-cluster-dns-and-coredns.md)  |
| ↳ Inspecting the CoreDNS Configuration: The Corefile | [33](notes/33-cluster-dns-and-coredns.md)  |
| ↳ CoreDNS Plugin Reference | [33](notes/33-cluster-dns-and-coredns.md)  |
| ↳ Pod DNS Resolution Modes in CoreDNS | [33](notes/33-cluster-dns-and-coredns.md)  |
| DNS Troubleshooting in Kubernetes | [33](notes/33-cluster-dns-and-coredns.md)  |
| ↳ 1. Verify CoreDNS is Running | [33](notes/33-cluster-dns-and-coredns.md)  |
| ↳ 2. Check CoreDNS Logs | [33](notes/33-cluster-dns-and-coredns.md)  |
| ↳ 3. Test DNS from Within a Pod | [33](notes/33-cluster-dns-and-coredns.md)  |
| ↳ 4. Verify resolv.conf Configuration | [33](notes/33-cluster-dns-and-coredns.md)  |
| ↳ 5. Check nsswitch.conf | [33](notes/33-cluster-dns-and-coredns.md)  |
| ↳ 6. Inspect CoreDNS ConfigMap (Corefile) | [33](notes/33-cluster-dns-and-coredns.md)  |
| ↳ 7. Check for Network Policy or Firewall Rules | [33](notes/33-cluster-dns-and-coredns.md)  |
| ↳ 8. Validate kubelet DNS Options | [33](notes/33-cluster-dns-and-coredns.md)  |
| ↳ 9. Look for Loop or Forwarding Errors | [33](notes/33-cluster-dns-and-coredns.md)  |
| Conclusion | [33](notes/33-cluster-dns-and-coredns.md)  |
| References | [33](notes/33-cluster-dns-and-coredns.md)  |

| File | Used in |
|---|---|
| `kind-cluster.yaml` | [33](notes/33-cluster-dns-and-coredns.md)  |

### Day 49 · MASTER Kubernetes Ingress | PART 1 | What, Why & Real-World Flow of Ingress | CKA 2025

| Original section | New home |
|---|---|
| Video reference for Day 49 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Introduction | [69](notes/69-ingress-concepts.md)  |
| Why Ingress? | [69](notes/69-ingress-concepts.md)  |
| What Are We Looking For? | [69](notes/69-ingress-concepts.md)  |
| What Is Ingress? | [69](notes/69-ingress-concepts.md)  |
| What Is an Ingress Controller? | [69](notes/69-ingress-concepts.md)  |
| Real-World Flow: How Ingress Works | [69](notes/69-ingress-concepts.md)  |
| Ingress Setup: Cloud-Native vs 3rd-Party Controllers | [69](notes/69-ingress-concepts.md)  |
| Popular Ingress Controllers | [69](notes/69-ingress-concepts.md)  |
| What’s Ahead | [69](notes/69-ingress-concepts.md)  |
| Conclusion | [69](notes/69-ingress-concepts.md)  |
| References | [69](notes/69-ingress-concepts.md)  |

| Image | Used in |
|---|---|
| `49a.png` | [69](notes/69-ingress-concepts.md)  |

### Day 50 · MASTER Kubernetes Ingress | PART 2 | Path-Based Routing on Amazon EKS with AWS ALB | CKA 2025

| Original section | New home |
|---|---|
| Video reference for Day 50 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Introduction | [70](notes/70-ingress-on-eks-path-routing.md)  |
| Demo 1: Path-Based Routing Using Ingress on Amazon EKS | [70](notes/70-ingress-on-eks-path-routing.md)  |
| ↳ What Are We Going to Deploy? | [70](notes/70-ingress-on-eks-path-routing.md)  |
| ↳ Real-World Ingress Routing: /iphone, /android, and the Desktop Default | [70](notes/70-ingress-on-eks-path-routing.md)  |
| ↳ Cluster Configuration | [70](notes/70-ingress-on-eks-path-routing.md)  |
| ↳ Prerequisites | [70](notes/70-ingress-on-eks-path-routing.md)  |
| Step 1: Provision EKS Cluster | [70](notes/70-ingress-on-eks-path-routing.md)  |
| Step 2: Install AWS Load Balancer Controller | [70](notes/70-ingress-on-eks-path-routing.md)  |
| ↳ 2.1 Create IAM Policy | [70](notes/70-ingress-on-eks-path-routing.md)  |
| ↳ 2.2 Create an IAM-Backed Kubernetes Service Account | [70](notes/70-ingress-on-eks-path-routing.md)  |
| ↳ 2.3 Install the AWS Load Balancer Controller Using Helm | [70](notes/70-ingress-on-eks-path-routing.md)  |
| Step 3: Create Deployments and Services for iPhone, Android, and Desktop Users | [70](notes/70-ingress-on-eks-path-routing.md)  |
| ↳ 3.1 Create a Dedicated Namespace | [70](notes/70-ingress-on-eks-path-routing.md)  |
| ↳ 3.2 iPhone Deployment and Service | [70](notes/70-ingress-on-eks-path-routing.md)  |
| ↳ 3.3 Android Deployment and Service | [70](notes/70-ingress-on-eks-path-routing.md)  |
| ↳ 3.4 Desktop Deployment and Service | [70](notes/70-ingress-on-eks-path-routing.md)  |
| ↳ Verification Commands | [70](notes/70-ingress-on-eks-path-routing.md)  |
| Step 4: Creating the Ingress Resource | [70](notes/70-ingress-on-eks-path-routing.md)  |
| ↳ Manifest: 05-ingress.yaml | [70](notes/70-ingress-on-eks-path-routing.md)  |
| Understanding ALB Target Types in Amazon EKS | [70](notes/70-ingress-on-eks-path-routing.md)  |
| ↳ target-type: ip — Direct-to-Pod Routing via VPC CNI | [70](notes/70-ingress-on-eks-path-routing.md)  |
| ↳ target-type: instance — NodePort-Based Routing via kube-proxy | [70](notes/70-ingress-on-eks-path-routing.md)  |
| ↳ Understanding IngressClass | [70](notes/70-ingress-on-eks-path-routing.md)  |
| ↳ When and Why You Might Run Multiple Ingress Controllers in a Cluster | [70](notes/70-ingress-on-eks-path-routing.md)  |
| ↳ Step 5: End-to-End Verification | [70](notes/70-ingress-on-eks-path-routing.md)  |
| ↳ Test the Routing | [70](notes/70-ingress-on-eks-path-routing.md)  |
| ↳ Verify the Ingress Configuration | [70](notes/70-ingress-on-eks-path-routing.md)  |
| ↳ Verify from the AWS Console | [70](notes/70-ingress-on-eks-path-routing.md)  |
| ↳ Step 6: Cleanup | [70](notes/70-ingress-on-eks-path-routing.md)  |
| Conclusion | [70](notes/70-ingress-on-eks-path-routing.md)  |
| References | [70](notes/70-ingress-on-eks-path-routing.md)  |

| Image | Used in |
|---|---|
| `49b.png` | [70](notes/70-ingress-on-eks-path-routing.md)  |
| `49c.png` | [70](notes/70-ingress-on-eks-path-routing.md)  |
| `49d.png` | [70](notes/70-ingress-on-eks-path-routing.md)  |
| `49e.png` | [70](notes/70-ingress-on-eks-path-routing.md)  |

| File | Used in |
|---|---|
| `demo1/01-ns.yaml` | [70](notes/70-ingress-on-eks-path-routing.md)  |
| `demo1/02-iphone.yaml` | [70](notes/70-ingress-on-eks-path-routing.md)  |
| `demo1/03-android.yaml` | [70](notes/70-ingress-on-eks-path-routing.md)  |
| `demo1/04-desktop.yaml` | [70](notes/70-ingress-on-eks-path-routing.md)  |
| `demo1/05-ingress.yaml` | [70](notes/70-ingress-on-eks-path-routing.md)  |
| `eks-config.yaml` | [70](notes/70-ingress-on-eks-path-routing.md)  |
| `iam_policy.json` | [70](notes/70-ingress-on-eks-path-routing.md)  |

### Day 51 · MASTER Kubernetes Ingress | PART 3 | TLS & Subdomain Routing on EKS with AWS ALB | CKA 2025

| Original section | New home |
|---|---|
| Video reference for Day 51 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Introduction | [71](notes/71-ingress-tls-and-host-routing.md)  |
| Pre-requisites | [71](notes/71-ingress-tls-and-host-routing.md)  |
| ↳ 1. Understand DNS Resolution | [71](notes/71-ingress-tls-and-host-routing.md)  |
| ↳ 2. Watch Ingress Part 1 – Real-World Flow of Ingress | [71](notes/71-ingress-tls-and-host-routing.md)  |
| ↳ 3. Watch Ingress Part 2 – Path-Based Routing on EKS | [71](notes/71-ingress-tls-and-host-routing.md)  |
| Demo 2: Ingress with TLS and Custom Domain Integration on Amazon EKS | [71](notes/71-ingress-tls-and-host-routing.md)  |
| Step 1: Register Domain and Request TLS Certificate | [71](notes/71-ingress-tls-and-host-routing.md)  |
| Step 2: Update Ingress Resource to Enable HTTPS | [71](notes/71-ingress-tls-and-host-routing.md)  |
| ↳ Step 3: Create a Route 53 Record | [71](notes/71-ingress-tls-and-host-routing.md)  |
| Step 4: Verification | [71](notes/71-ingress-tls-and-host-routing.md)  |
| ↳ Step 5: Cleanup | [71](notes/71-ingress-tls-and-host-routing.md)  |
| Demo 3: Name-Based Routing Using ALB Ingress on EKS | [71](notes/71-ingress-tls-and-host-routing.md)  |
| Step 1: Request Certificates in ACM | [71](notes/71-ingress-tls-and-host-routing.md)  |
| Step 2: Update Deployments and Services | [71](notes/71-ingress-tls-and-host-routing.md)  |
| ↳ Key Changes | [71](notes/71-ingress-tls-and-host-routing.md)  |
| ↳ Sample Manifest for iPhone Service | [71](notes/71-ingress-tls-and-host-routing.md)  |
| Step 3: Create Ingress for Name-Based Routing | [71](notes/71-ingress-tls-and-host-routing.md)  |
| ↳ Explanation | [71](notes/71-ingress-tls-and-host-routing.md)  |
| Step 4: Route 53 Alias Records | [71](notes/71-ingress-tls-and-host-routing.md)  |
| Step 5: Verification | [71](notes/71-ingress-tls-and-host-routing.md)  |
| Step 6: Cleanup | [71](notes/71-ingress-tls-and-host-routing.md)  |
| Conclusion | [71](notes/71-ingress-tls-and-host-routing.md)  |
| References | [71](notes/71-ingress-tls-and-host-routing.md)  |
| ↳ Kubernetes Official Documentation | [71](notes/71-ingress-tls-and-host-routing.md)  |
| ↳ AWS Official Documentation | [71](notes/71-ingress-tls-and-host-routing.md)  |

| Image | Used in |
|---|---|
| `51a.png` | [71](notes/71-ingress-tls-and-host-routing.md)  |
| `51b.png` | [71](notes/71-ingress-tls-and-host-routing.md)  |

| File | Used in |
|---|---|
| `demo2/01-ns.yaml` | [71](notes/71-ingress-tls-and-host-routing.md)  |
| `demo2/02-iphone.yaml` | [71](notes/71-ingress-tls-and-host-routing.md)  |
| `demo2/03-android.yaml` | [71](notes/71-ingress-tls-and-host-routing.md)  |
| `demo2/04-desktop.yaml` | [71](notes/71-ingress-tls-and-host-routing.md)  |
| `demo2/05-ingress.yaml` | [71](notes/71-ingress-tls-and-host-routing.md)  |
| `demo3/01-ns.yaml` | [71](notes/71-ingress-tls-and-host-routing.md)  |
| `demo3/02-iphone.yaml` | [71](notes/71-ingress-tls-and-host-routing.md)  |
| `demo3/03-android.yaml` | [71](notes/71-ingress-tls-and-host-routing.md)  |
| `demo3/04-desktop.yaml` | [71](notes/71-ingress-tls-and-host-routing.md)  |
| `demo3/05-ingress.yaml` | [71](notes/71-ingress-tls-and-host-routing.md)  |
| `eks-config.yaml` | [71](notes/71-ingress-tls-and-host-routing.md)  |
| `iam_policy.json` | [71](notes/71-ingress-tls-and-host-routing.md)  |

### Day 52 · Kubernetes Gateway API Deep Dive | Part 1 | GatewayClass, Gateway & Routes | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 52 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Introduction | [72](notes/72-gateway-api-concepts.md)  |
| Pre-Requisites for Gateway API Lectures | [72](notes/72-gateway-api-concepts.md)  |
| ↳ Ingress Series | [72](notes/72-gateway-api-concepts.md)  |
| ↳ Custom Resources & Operators | [72](notes/72-gateway-api-concepts.md)  |
| What We Already Know About Ingress | [72](notes/72-gateway-api-concepts.md)  |
| ↳ 1. Ingress is Native but Minimal | [72](notes/72-gateway-api-concepts.md)  |
| ↳ 2. Advanced Features Come from the Controller | [72](notes/72-gateway-api-concepts.md)  |
| ↳ 3. Controller-Specific Annotations Are Common Practice | [72](notes/72-gateway-api-concepts.md)  |
| What’s Missing in Ingress | [72](notes/72-gateway-api-concepts.md)  |
| ↳ 1. Cross-Namespace, Multi-Tenancy & RBAC | [72](notes/72-gateway-api-concepts.md)  |
| ↳ 2. Protocol & Feature Scope | [72](notes/72-gateway-api-concepts.md)  |
| ↳ 3. Controller-Specific Annotations and Lack of Validation | [72](notes/72-gateway-api-concepts.md)  |
| What is Gateway API | [72](notes/72-gateway-api-concepts.md)  |
| Gateway API Resource Model and Role Alignment | [72](notes/72-gateway-api-concepts.md)  |
| ↳ Core Resource Types | [72](notes/72-gateway-api-concepts.md)  |
| Protocol Flexibility and Advanced Traffic Management | [72](notes/72-gateway-api-concepts.md)  |
| Gateway API Flow: Controller, GatewayClass, Gateway, and HTTPRoute | [72](notes/72-gateway-api-concepts.md)  |
| ↳ 1. Controller Watches Gateways and Routes | [72](notes/72-gateway-api-concepts.md)  |
| ↳ 2. GatewayClass Defines LB Type and Controller | [72](notes/72-gateway-api-concepts.md)  |
| ↳ 3. Gateway Implements Listener and Traffic Forwarding | [72](notes/72-gateway-api-concepts.md)  |
| ↳ 4. Data Plane | [72](notes/72-gateway-api-concepts.md)  |
| ↳ 5. HTTPRoute Defines Routing Rules | [72](notes/72-gateway-api-concepts.md)  |
| ↳ One-Line Flows for Quick Recall | [72](notes/72-gateway-api-concepts.md)  |
| Gateway API Setup: Cloud-Managed vs In-Cluster Data Planes | [72](notes/72-gateway-api-concepts.md)  |
| ↳ Key Differences | [72](notes/72-gateway-api-concepts.md)  |
| Conclusion | [72](notes/72-gateway-api-concepts.md)  |
| References | [72](notes/72-gateway-api-concepts.md)  |

| Image | Used in |
|---|---|
| `52a.png` | [72](notes/72-gateway-api-concepts.md)  |
| `52b.png` | [72](notes/72-gateway-api-concepts.md)  |
| `52c.png` | [72](notes/72-gateway-api-concepts.md)  |

### Day 53 · Kubernetes Gateway API Demo | Part 2 | NGINX Gateway Fabric | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 53 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Introduction | [73](notes/73-gateway-api-hands-on.md)  |
| Pre-Requisite | [73](notes/73-gateway-api-hands-on.md)  |
| Demo Introduction | [73](notes/73-gateway-api-hands-on.md)  |
| What We’re Going to Do in This Demo | [73](notes/73-gateway-api-hands-on.md)  |
| Flow | [73](notes/73-gateway-api-hands-on.md)  |
| Pre-requisites | [73](notes/73-gateway-api-hands-on.md)  |
| Step 1: Configure KIND Cluster | [73](notes/73-gateway-api-hands-on.md)  |
| ↳ 1.1 Delete any existing cluster | [73](notes/73-gateway-api-hands-on.md)  |
| ↳ 1.2 KIND cluster config (00-kind-cluster.yaml) | [73](notes/73-gateway-api-hands-on.md)  |
| ↳ 1.3 Create the cluster | [73](notes/73-gateway-api-hands-on.md)  |
| Step 2: Install Gateway API CRDs | [73](notes/73-gateway-api-hands-on.md)  |
| ↳ ⚠️ Important note on versions (read carefully) | [73](notes/73-gateway-api-hands-on.md)  |
| ↳ What gets installed (1-liners) | [73](notes/73-gateway-api-hands-on.md)  |
| ↳ Verify installation | [73](notes/73-gateway-api-hands-on.md)  |
| Step 3: Install the NGINX Gateway Fabric Controller | [73](notes/73-gateway-api-hands-on.md)  |
| Step 4: Deploy Application Manifests | [73](notes/73-gateway-api-hands-on.md)  |
| ↳ 4.1 iPhone App (02-iphone.yaml) | [73](notes/73-gateway-api-hands-on.md)  |
| ↳ 4.2 Android App (03-android.yaml) | [73](notes/73-gateway-api-hands-on.md)  |
| ↳ 4.3 Desktop App (04-desktop.yaml) | [73](notes/73-gateway-api-hands-on.md)  |
| Step 5: Create the Gateway | [73](notes/73-gateway-api-hands-on.md)  |
| ↳ Apply | [73](notes/73-gateway-api-hands-on.md)  |
| ↳ Verify (controller picked it up, data plane exposed) | [73](notes/73-gateway-api-hands-on.md)  |
| ↳ What happens next | [73](notes/73-gateway-api-hands-on.md)  |
| Step 6 – Create HTTPRoutes | [73](notes/73-gateway-api-hands-on.md)  |
| ↳ Apply | [73](notes/73-gateway-api-hands-on.md)  |
| ↳ Verify | [73](notes/73-gateway-api-hands-on.md)  |
| ↳ Notes on Route Types | [73](notes/73-gateway-api-hands-on.md)  |
| Step 7 – Check Connectivity | [73](notes/73-gateway-api-hands-on.md)  |
| ↳ Test Locally via NodePort | [73](notes/73-gateway-api-hands-on.md)  |
| ↳ Expected Output | [73](notes/73-gateway-api-hands-on.md)  |
| ↳ Validate Routing via Gateway Proxy Pod | [73](notes/73-gateway-api-hands-on.md)  |
| Conclusion | [73](notes/73-gateway-api-hands-on.md)  |
| References | [73](notes/73-gateway-api-hands-on.md)  |

| Image | Used in |
|---|---|
| `53a.png` | [73](notes/73-gateway-api-hands-on.md)  |

| File | Used in |
|---|---|
| `demo/00-kind-cluster.yaml` | [73](notes/73-gateway-api-hands-on.md)  |
| `demo/01-ns.yaml` | [73](notes/73-gateway-api-hands-on.md)  |
| `demo/02-iphone.yaml` | [73](notes/73-gateway-api-hands-on.md)  |
| `demo/03-android.yaml` | [73](notes/73-gateway-api-hands-on.md)  |
| `demo/04-desktop.yaml` | [73](notes/73-gateway-api-hands-on.md)  |
| `demo/05-gateway.yaml` | [73](notes/73-gateway-api-hands-on.md)  |
| `demo/06-routes.yaml` | [73](notes/73-gateway-api-hands-on.md)  |

### Day 54 · Build a Multi-Node Kubernetes Cluster with kubeadm + Calico Operator | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 54 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Lecture Prerequisites | [75](notes/75-kubeadm-cluster-build.md)  |
| Demo Pre-requisites | [75](notes/75-kubeadm-cluster-build.md)  |
| ↳ 1) Networking & Security Groups (updated for Calico Typha) | [75](notes/75-kubeadm-cluster-build.md)  |
| ↳ 2) SSH key pair | [75](notes/75-kubeadm-cluster-build.md)  |
| ↳ 3) Instances | [75](notes/75-kubeadm-cluster-build.md)  |
| ↳ 4) Hostnames (clarity only) | [75](notes/75-kubeadm-cluster-build.md)  |
| ↳ 5) How to follow the steps (read this first) | [75](notes/75-kubeadm-cluster-build.md)  |
| Step 1: Disable swap & set kernel networking \[ALL NODES] | [75](notes/75-kubeadm-cluster-build.md)  |
| Step 2: Install and configure containerd \[ALL NODES] | [75](notes/75-kubeadm-cluster-build.md)  |
| Step 3: Install kubeadm, kubelet, kubectl (v1.32) \[ALL NODES] | [75](notes/75-kubeadm-cluster-build.md)  |
| ↳ Bash completion (optional, helpful) | [75](notes/75-kubeadm-cluster-build.md)  |
| Step 4: Initialize the control plane \[CONTROL PLANE ONLY] | [75](notes/75-kubeadm-cluster-build.md)  |
| Step 5: Install Calico CNI via Operator (defaults) \[CONTROL PLANE ONLY] | [75](notes/75-kubeadm-cluster-build.md)  |
| Step 6: Join the workers \[WORKER ONLY] | [75](notes/75-kubeadm-cluster-build.md)  |
| Step 7: Verify & quick demo \[any node with kubeconfig] | [75](notes/75-kubeadm-cluster-build.md)  |
| Calico Troubleshooting | [75](notes/75-kubeadm-cluster-build.md)  |
| Fix (VXLAN clusters) | [75](notes/75-kubeadm-cluster-build.md)  |
| Post-install reboots & stabilization (strongly recommended) | [75](notes/75-kubeadm-cluster-build.md)  |
| Conclusion | [75](notes/75-kubeadm-cluster-build.md)  |
| References | [75](notes/75-kubeadm-cluster-build.md)  |

| Image | Used in |
|---|---|
| `54a.png` | [75](notes/75-kubeadm-cluster-build.md)  |

### Day 55 · Upgrade a Multi-Node Kubernetes Cluster with kubeadm | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 55 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Introduction | [76](notes/76-versions-skew-and-upgrades.md)  |
| Versioning & Release Cadence | [76](notes/76-versions-skew-and-upgrades.md)  |
| Support Window & Real-World Practice | [76](notes/76-versions-skew-and-upgrades.md)  |
| Patch Management Guideline (N / N-1 / N-2) | [76](notes/76-versions-skew-and-upgrades.md)  |
| Skipping Versions & Sequencing | [76](notes/76-versions-skew-and-upgrades.md)  |
| Version Skew (Component Compatibility) | [76](notes/76-versions-skew-and-upgrades.md)  |
| Why Upgrades Matter | [76](notes/76-versions-skew-and-upgrades.md)  |
| High-Level Upgrade Flow (Production) | [76](notes/76-versions-skew-and-upgrades.md)  |
| Upgrade Strategies | [76](notes/76-versions-skew-and-upgrades.md)  |
| Scope & Assumptions | [76](notes/76-versions-skew-and-upgrades.md)  |
| Pre-Requisites (Read Once) | [76](notes/76-versions-skew-and-upgrades.md)  |
| Control-Plane Upgrade | [76](notes/76-versions-skew-and-upgrades.md)  |
| ↳ Step 1: Point APT to the v1.33 repo | [76](notes/76-versions-skew-and-upgrades.md)  |
| ↳ Step 2: Determine the exact target version | [76](notes/76-versions-skew-and-upgrades.md)  |
| ↳ Step 3: Upgrade kubeadm (control plane) | [76](notes/76-versions-skew-and-upgrades.md)  |
| ↳ Step 4: Plan the upgrade (dry-run) | [76](notes/76-versions-skew-and-upgrades.md)  |
| ↳ Step 5: Apply the control-plane upgrade | [76](notes/76-versions-skew-and-upgrades.md)  |
| ↳ Step 6: Verify server version and node skew | [76](notes/76-versions-skew-and-upgrades.md)  |
| ↳ Step 7: Upgrade node-local bits on the control-plane node | [76](notes/76-versions-skew-and-upgrades.md)  |
| Worker Node Upgrades (one at a time) | [76](notes/76-versions-skew-and-upgrades.md)  |
| ↳ Step 1: Cordon, then drain (from control plane or any admin shell) | [76](notes/76-versions-skew-and-upgrades.md)  |
| ↳ Step 2: Upgrade the worker (on the worker node) | [76](notes/76-versions-skew-and-upgrades.md)  |
| ↳ Step 3 — Uncordon and verify (from admin shell) | [76](notes/76-versions-skew-and-upgrades.md)  |
| Calico Post-Upgrade Troubleshooting | [76](notes/76-versions-skew-and-upgrades.md)  |
| Fix (VXLAN clusters) | [76](notes/76-versions-skew-and-upgrades.md)  |
| Draining with a PodDisruptionBudget (What the diagram shows) | [76](notes/76-versions-skew-and-upgrades.md)  |
| Conclusion | [76](notes/76-versions-skew-and-upgrades.md)  |
| References | [76](notes/76-versions-skew-and-upgrades.md)  |

| Image | Used in |
|---|---|
| `55a.png` | [76](notes/76-versions-skew-and-upgrades.md)  |

| File | Used in |
|---|---|
| `pdb.yaml` | [76](notes/76-versions-skew-and-upgrades.md)  |

### Day 56 · Kubernetes Monitoring & Logging Explained | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 56 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Introduction | [79](notes/79-monitoring-and-logging-basics.md)  |
| ↳ 1. What is Observability? | [79](notes/79-monitoring-and-logging-basics.md)  |
| ↳ 2. Two Perspectives of Observability (DevOps View) | [79](notes/79-monitoring-and-logging-basics.md)  |
| Observability in AWS with Native Services | [79](notes/79-monitoring-and-logging-basics.md)  |
| ↳ 3. The 3 Pillars of Observability | [79](notes/79-monitoring-and-logging-basics.md)  |
| Metrics Server — Quick Install (Lab) | [48](notes/48-metrics-server-and-hpa.md)  |
| ↳ Step 1 — Install | [48](notes/48-metrics-server-and-hpa.md)  |
| ↳ Step 2 — Edit the Deployment | [48](notes/48-metrics-server-and-hpa.md)  |
| ↳ Step 3 — Verify | [48](notes/48-metrics-server-and-hpa.md)  |
| If you used Day 54 (kubeadm + Calico Operator) | [48](notes/48-metrics-server-and-hpa.md)  |
| Out-of-the-Box Monitoring and Logging in Kubernetes | [79](notes/79-monitoring-and-logging-basics.md)  |
| ↳ Kubernetes Monitoring | [79](notes/79-monitoring-and-logging-basics.md)  |
| ↳ Metrics Server architecture (what happens on kubectl top) | [79](notes/79-monitoring-and-logging-basics.md)  |
| ↳ What you get | [79](notes/79-monitoring-and-logging-basics.md)  |
| ↳ Key notes / gotchas | [79](notes/79-monitoring-and-logging-basics.md)  |
| ↳ Beyond Metrics Server (add-ons & native signals) | [79](notes/79-monitoring-and-logging-basics.md)  |
| ↳ Kubernetes Logging | [79](notes/79-monitoring-and-logging-basics.md)  |
| Conclusion | [79](notes/79-monitoring-and-logging-basics.md)  |
| References | [79](notes/79-monitoring-and-logging-basics.md)  |

| Image | Used in |
|---|---|
| `56a.png` | [79](notes/79-monitoring-and-logging-basics.md)  |
| `56b.png` | [79](notes/79-monitoring-and-logging-basics.md)  |
| `56c.png` | [79](notes/79-monitoring-and-logging-basics.md)  |
| `56d.png` | [79](notes/79-monitoring-and-logging-basics.md)  |

| File | Used in |
|---|---|
| `01-demo-pod.yaml` | [79](notes/79-monitoring-and-logging-basics.md)  |

### Day 57 · Kubernetes Control Plane Troubleshooting Guide | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 57 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Introduction | [82](notes/82-control-plane-troubleshooting.md)  |
| ↳ Prerequisites for this lecture | [82](notes/82-control-plane-troubleshooting.md)  |
| Control-Plane Basics (kubeadm clusters) | [82](notes/82-control-plane-troubleshooting.md)  |
| kubeconfig & contexts (quick refresher) | [82](notes/82-control-plane-troubleshooting.md)  |
| 1) API server unreachable / kubectl times out | [82](notes/82-control-plane-troubleshooting.md)  |
| Tooling primer: ss and the flags you’re using | [82](notes/82-control-plane-troubleshooting.md)  |
| ↳ Reading ss -tlpns (listening TCP sockets) | [82](notes/82-control-plane-troubleshooting.md)  |
| ↳ Local vs Peer (quick mental model) | [82](notes/82-control-plane-troubleshooting.md)  |
| Established connections (remove -l) | [82](notes/82-control-plane-troubleshooting.md)  |
| 2) etcd unhealthy → API server crashloop | [82](notes/82-control-plane-troubleshooting.md)  |
| ↳ Likely causes → How to fix | [82](notes/82-control-plane-troubleshooting.md)  |
| ↳ Quick checks (with what they tell you) | [82](notes/82-control-plane-troubleshooting.md)  |
| ↳ Symptom clues in logs (to pinpoint root cause) | [82](notes/82-control-plane-troubleshooting.md)  |
| 3) Pods stay Pending — scheduler unavailable *or* no feasible node | [82](notes/82-control-plane-troubleshooting.md)  |
| ↳ Fast triage (20 seconds) | [82](notes/82-control-plane-troubleshooting.md)  |
| ↳ A) Scheduler unavailable | [82](notes/82-control-plane-troubleshooting.md)  |
| ↳ B) Scheduler running but can’t place the Pod | [82](notes/82-control-plane-troubleshooting.md)  |
| 4) Controller-Manager down → controllers inactive (no replicas, no GC) | [82](notes/82-control-plane-troubleshooting.md)  |
| ↳ Likely causes | [82](notes/82-control-plane-troubleshooting.md)  |
| ↳ Checks (what each tells you) | [82](notes/82-control-plane-troubleshooting.md)  |
| ↳ Fix | [82](notes/82-control-plane-troubleshooting.md)  |
| 5) Certificates expired (very common exam task) | [82](notes/82-control-plane-troubleshooting.md)  |
| ↳ Quick Checks (what’s wrong?) | [82](notes/82-control-plane-troubleshooting.md)  |
| ↳ Break (demo) — simulate expiry by jumping time | [82](notes/82-control-plane-troubleshooting.md)  |
| ↳ Fix (exam way) — rotate with kubeadm, refresh kubeconfigs | [82](notes/82-control-plane-troubleshooting.md)  |
| ↳ Optional cleanup — restore real time then re-renew (important!) | [82](notes/82-control-plane-troubleshooting.md)  |
| After cert renewal: controller-manager & scheduler NotReady | [82](notes/82-control-plane-troubleshooting.md)  |
| ↳ Why this happens | [82](notes/82-control-plane-troubleshooting.md)  |
| ↳ Fixing controller-manager | [82](notes/82-control-plane-troubleshooting.md)  |
| ↳ Fixing scheduler | [82](notes/82-control-plane-troubleshooting.md)  |
| ↳ Also double-check (common after cert work) | [82](notes/82-control-plane-troubleshooting.md)  |
| ↳ TL;DR | [82](notes/82-control-plane-troubleshooting.md)  |
| Conclusion | [82](notes/82-control-plane-troubleshooting.md)  |
| References | [82](notes/82-control-plane-troubleshooting.md)  |

### Day 58 · Kubernetes Data Plane Troubleshooting Guide | CKA Course 2025

| Original section | New home |
|---|---|
| Video reference for Day 58 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Introduction | [83](notes/83-data-plane-troubleshooting.md)  |
| ↳ Prerequisites for this lecture | [83](notes/83-data-plane-troubleshooting.md)  |
| ↳ Quick refresher: the data-plane pieces | [83](notes/83-data-plane-troubleshooting.md)  |
| Read This First: How to Use the Scenarios | [21](notes/21-debugging-pods.md)  |
| 1) Node NotReady / kubelet not registered | [83](notes/83-data-plane-troubleshooting.md)  |
| ↳ Likely causes → How to fix | [83](notes/83-data-plane-troubleshooting.md)  |
| ↳ Quick checks (with what they tell you) | [83](notes/83-data-plane-troubleshooting.md)  |
| ↳ Symptom clues in logs (what to look for) | [83](notes/83-data-plane-troubleshooting.md)  |
| 2) Pods stuck ContainerCreating / CNI not initialized | [29](notes/29-network-model-and-cni.md)  |
| ↳ Likely causes → How to fix | [29](notes/29-network-model-and-cni.md)  |
| ↳ Checks (quick and surgical) | [29](notes/29-network-model-and-cni.md)  |
| ↳ Fix (minimal moves, then verify) | [29](notes/29-network-model-and-cni.md)  |
| 3) Pod-to-Pod across nodes / Service ClusterIP not reachable | [32](notes/32-inside-a-service-kube-proxy.md)  |
| ↳ Likely causes → How to fix | [32](notes/32-inside-a-service-kube-proxy.md)  |
| ↳ Checks (quick) | [32](notes/32-inside-a-service-kube-proxy.md)  |
| ↳ Fix (minimal moves, then verify) | [32](notes/32-inside-a-service-kube-proxy.md)  |
| 4) DNS failures (CoreDNS) | [33](notes/33-cluster-dns-and-coredns.md)  |
| 5) ImagePullBackOff / ErrImagePull | [21](notes/21-debugging-pods.md)  |
| ↳ Likely causes → How to fix | [21](notes/21-debugging-pods.md)  |
| ↳ Checks (quick) | [21](notes/21-debugging-pods.md)  |
| ↳ Fix (minimal, then verify) | [21](notes/21-debugging-pods.md)  |
| 6) CrashLoopBackOff / OOMKilled / probe failures | [21](notes/21-debugging-pods.md)  |
| ↳ Likely causes → How to fix | [21](notes/21-debugging-pods.md)  |
| ↳ Checks (quick) | [21](notes/21-debugging-pods.md)  |
| ↳ Fix (minimal, then verify) | [21](notes/21-debugging-pods.md)  |
| 7) PVC Pending / volume mount errors (CSI) | [83](notes/83-data-plane-troubleshooting.md)  |
| ↳ Likely causes → How to fix | [83](notes/83-data-plane-troubleshooting.md)  |
| ↳ Checks (quick) | [83](notes/83-data-plane-troubleshooting.md)  |
| ↳ Fix (minimal, then verify) | [83](notes/83-data-plane-troubleshooting.md)  |
| 8) NodePort / LoadBalancer not reachable | [83](notes/83-data-plane-troubleshooting.md)  |
| ↳ Likely causes → How to fix | [83](notes/83-data-plane-troubleshooting.md)  |
| ↳ Checks (quick) | [83](notes/83-data-plane-troubleshooting.md)  |
| ↳ Fix (minimal, then verify) | [83](notes/83-data-plane-troubleshooting.md)  |
| ↳ Conclusion | [83](notes/83-data-plane-troubleshooting.md)  |
| ↳ References | [83](notes/83-data-plane-troubleshooting.md)  |

### Day 59 · Kubernetes JSONPath with kubectl | CKA 2025

| Original section | New home |
|---|---|
| Video reference for Day 59 is the following: | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| ⭐ Support the Project | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Table of Contents | Not carried over (video link / repo star request / table of contents: navigation only). It is still in the unchanged original folder |
| Introduction | [16](notes/16-kubectl-output-jsonpath-and-friends.md)  |
| ↳ Prerequisites | [16](notes/16-kubectl-output-jsonpath-and-friends.md)  |
| Behind the Scenes of kubectl get pods (Why It Matters for JSONPath) | [16](notes/16-kubectl-output-jsonpath-and-friends.md) [14](notes/14-the-kubernetes-api.md)  |
| ↳ Why this matters before we start JSONPath | [16](notes/16-kubectl-output-jsonpath-and-friends.md) [14](notes/14-the-kubernetes-api.md)  |
| ↳ YAML ↔ JSON in Kubernetes | [16](notes/16-kubectl-output-jsonpath-and-friends.md) [14](notes/14-the-kubernetes-api.md)  |
| Why JSONPath? | [16](notes/16-kubectl-output-jsonpath-and-friends.md)  |
| What is JSONPath? | [16](notes/16-kubectl-output-jsonpath-and-friends.md)  |
| ↳ Core operators (you’ll use these 90% of the time) | [16](notes/16-kubectl-output-jsonpath-and-friends.md)  |
| ↳ Kubernetes-flavored examples | [16](notes/16-kubectl-output-jsonpath-and-friends.md)  |
| Practice locally first (single file) | [16](notes/16-kubectl-output-jsonpath-and-friends.md)  |
| ↳ Basics (why start from $?) | [16](notes/16-kubectl-output-jsonpath-and-friends.md)  |
| ↳ Containers (arrays & wildcards) | [16](notes/16-kubectl-output-jsonpath-and-friends.md)  |
| ↳ Indexing & filtering | [16](notes/16-kubectl-output-jsonpath-and-friends.md)  |
| Using JSONPath with kubectl (tight, practical) | [16](notes/16-kubectl-output-jsonpath-and-friends.md)  |
| ↳ Essentials | [16](notes/16-kubectl-output-jsonpath-and-friends.md)  |
| ↳ Apply on live objects (with annotated JSONPath) | [16](notes/16-kubectl-output-jsonpath-and-friends.md)  |
| ↳ Pro tips | [16](notes/16-kubectl-output-jsonpath-and-friends.md)  |
| References | [16](notes/16-kubectl-output-jsonpath-and-friends.md)  |

| Image | Used in |
|---|---|
| `59a.png` | [16](notes/16-kubectl-output-jsonpath-and-friends.md)  |

| File | Used in |
|---|---|
| `pod.json` | [16](notes/16-kubectl-output-jsonpath-and-friends.md)  |

