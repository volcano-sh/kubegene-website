# volumes

Optional, information about volume that required gene sequencing process required, such as mount path in container, pvc name and so on. You can specify multiple volumes.

## Volumes format

```
volumes: 
  <volume name>:
    mountPath: <mountPath>
    mountFrom: <pvc info>
```

## field

<table>
   <tr>
      <td>field</td>
      <td>required</td>
      <td>type</td>
      <td>description</td>
   </tr>
   <tr>
      <td>mountPath</td>
      <td>Yes</td>
      <td>string</td>
      <td>Path within the container at which the volume should be mounted. Must not contain ':'.</td>
   </tr>
   <tr>
      <td>mountFrom</td>
      <td>Yes</td>
      <td>struct</td>
      <td>Detail info about volume</td>
   </tr>
</table>


mountFrom field description

<table>
   <tr>
      <td>field</td>
      <td>required</td>
      <td>type</td>
      <td>description</td>
   </tr>
   <tr>
      <td>pvc</td>
      <td>Yes</td>
      <td>string</td>
      <td>The pvc name. Note: the specified pvc must exist in the cluster.</td>
   </tr>
</table>


### volumes example
```
volumes:
  genref:
    mountPath: ${volume-path-ref}
    mountFrom:
      pvc: ${my_k8s_pvc}
  genobs:
    mountPath: /volume-path-obs
    mountFrom:
      pvc: sample-data-pvc
```