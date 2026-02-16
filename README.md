# kubernetes-dispatcharr-gluetun
playbook and var file for kubernetes playbook

At somepoint I'll take a look and split this up into multiple playbooks.  one for redis, one for postgres, one for the web/gluetun and one for celery.  I guess you can call this 1.0.  First.  This creates everything (redis,postgres,cerley,web/gluetun) in its own pod. I've also set this to prefer this isn't scheduled on the same node.  The playbook has 

    dispatcharr_spread_anti_affinity:
      podAntiAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
                - key: app.kubernetes.io/part-of
                  operator: In
                  values: ["dispatcharr"]
            topologyKey: kubernetes.io/hostname

    dispatcharr_topology_spread:
      - maxSkew: 1
        topologyKey: kubernetes.io/hostname
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            app.kubernetes.io/part-of: dispatcharr

This defines the schedular.   With this alone it'll still schedule everything on the same node.  adding the below to each metadata.template.spec you force it to schedule on separate nodes.  

    affinity: "{{ dispatcharr_spread_anti_affinity }}"
    topologySpreadConstraints: "{{ dispatcharr_topology_spread }}"

I wanted this as I didnt want any part of dispatcharr to affect another part.  I.E. Celery eating cpu which can affect streams or database functions.  You can remove this specs if you want.  Entirely up to you.


To install,  (the -J is assuming your vars file is encrypted with ansible-vault. If its not remove the -J  if you want verbose add -vvv after the -J)
ansible-playbook -i yourinventoryfile.yml dispatcharr-modular.yml -J             
