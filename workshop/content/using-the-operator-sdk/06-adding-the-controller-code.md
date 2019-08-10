In order for the operator to be able to respond to the custom resource, a controller for the resource type must be added to the operator code. To create the controller code run:

```execute
operator-sdk add controller --api-version=app.example.com/v1alpha1 --kind=PodSet
```

This will register the controller with the operator main program and set up a watch on the custom resource, such that when any changes are made, the reconciliation loop of the operator is invoked and any appropriate action can take place.

You can view the initial generated code for the controller by running:

```execute
cat pkg/controller/podset/podset_controller.go
```

The `add()` function in the generated code file is what sets up the watch for the `PodSet` custom resource.

```
// Watch for changes to primary resource PodSet
err = c.Watch(&source.Kind{Type: &appv1alpha1.PodSet{}}, &handler.EnqueueRequestForObject{})
```

The creation of new instances of the `PodSet` custom resource, or changes to existing instances of the custom resource, will result in the `Reconcile()` function being called. This is where your custom logic needs to be added to respond to any changes.

For a new instance of the `PodSet` custom resource, we want the operator to start up as many pods as the custom resource defines in the `spec.replicas` attribute.

In the case of an existing instance of the custom resource being updated, and the value of `spec.replicas` changed, the operator needs to correspondingly adjust the number of pods up or down.

If the custom resource is deleted, then all the pods need to be deleted.

In addition to the operator watching for changes to the custom resource, it also needs to monitor the pods corresponding to each instance of the custom resource. This is so that if a pod is shutdown, or was deleted, that a new pod is started in its place. In order to do this, the operator code also needs to watch instances of a `Pod`. This is also setup in the `add()` function.

```
// TODO(user): Modify this to be the types you create that are owned by the primary resource
// Watch for changes to secondary resource Pods and requeue the owner PodSet
err = c.Watch(&source.Kind{Type: &corev1.Pod{}}, &handler.EnqueueRequestForOwner{
        IsController: true,
        OwnerType:    &appv1alpha1.PodSet{},
})
```

If you look at the generated code for the controller, you will see this already exists. This is because the code provides as a sample, code for creating a single pod for each instance of a custom resource. When starting a pod, it is using the `busybox` image. The pod is created from the `newPodForCR()` function in this example.