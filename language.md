# Cellery Language Syntax

The Cellery Language is a subset of [Ballerina](http://ballerina.io) and Ballerina extensions. Therefore language syntax 
of Cellery, resembles with normal Ballerina language syntax. 

## Cell
A cell is a collection of [components](#component), [APIs](#api) and [Egresses](#egress). A Cell object is initialized 
as follows;
```
cellery:Cell cellObj = new;
```

Components can be added to a cell by calling `addComponent(Component comp)` method.
```
cellObj.addComponent(comp);
```

Parameters of the Cell object can be assigned/modified as follows;
```
cellObj.egresses = [
    {
        parent: component.name,
        ingress: component.ingresses[0],
        envVar: "ENV1"
    }
];
cellObj.apis = [
    {
        parent: componentB.name,
        context:componentB.ingresses[0],
        global: true
    }
];
```

## Component
A component represents an implementation of the business logic (in a docker image or a legacy system) and a collections 
of network accessible entry points ([Ingresses](#ingress)) and external communications ([Egresses](#egress)). 

A `component` is defined as follows:
```
cellery:Component component = {
    name: string,
    source : (ImageSource|DockerSource| GitSource),
    env?: map<string>,
    ingresses?: [{ingress}*],
    egresses?: [{egress}*]
};
```

`source`, `env`, `ingresses`, `egresses` can be defined inline or can refer to externally defined instances.

### Component Source
`source` of the component can be one of the following types. 

#### ImageSource
This is an already available docker image which is hosted in an accessible registry. Definition is as follows;
```
cellery:ImageSource imgSource = {
    image: "docker.io/cellery/component:v1"
};
```
#### DockerSource
This is referring to a Dockerfile which can be used to build a docker image. Definition is as follows;
```
cellery:DockerSource dockerSource = {
    Dockerfile: "/path/to/Dockerfile",
    tag: "component:v1"
};
```

#### GitSource
This is referring to a Git Repository which contains the resources (with correct references) to build a docker image. 
Definition is as follows;
 
```
cellery:GitSource gitSource = {
    gitRepo: "github.com/org/repo",
    tag: "component:v1"
};
```

A sample component with inline definitions would be as follows;
```
cellery:Component componentA = {
    name: "ComponentA",
    source: {
        image: "docker.io/wso2vick/component-a:v1"
    },
    env: { ENV1: "", ENV2: "" },
    ingresses: [
        {
            name: "foo",
            port: "8080:80",
            context: "foo",
            definitions: [
                {
                    path: "*",
                    method: "GET,POST,PUT,DELETE"
                }
            ]
        }
    ],
    egresses: [
        {
            parent: componentB.name,
            ingress: componentB.ingresses[0],
            envVar: "ENV1",
            resiliency: {
                retryConfig: {
                    interval: 100,
                    count: 10,
                    backOffFactor: 0.5,
                    maxWaitInterval: 20000
                }
            }
        }
    ]
};
```

A sample component with external instance definitions would be as follows;
```
cellery:RetryConfig retryConfig = {
    interval: 100,
    count: 10,
    backOffFactor: 0.5,
    maxWaitInterval: 20000
};

cellery:FailoverConfig failOverConfig = {
    endpoiint: "bar.com/failover",
    timeOut: 60
};

cellery:Resiliency resiliency = {
    retryConfig: retryConfig
};

cellery:ImageSource imgSourceA = {
    image: "docker.io/cellery/component-a:v1"
};

cellery:Ingress ingressA = {
    name: "foo",
    port: "8080:80",
    context: "foo",
    definitions: [
        {
            path: "*",
            method: "GET,POST,PUT,DELETE"
        }
    ]
};

cellery:Egress egressA = {
    parent: componentB.name,
    ingress: componentB.ingresses[0],
    envVar: "ENV1",
    resiliency: resiliency
};


cellery:Component componentA = {
    name: "ComponentA",
    source: imgSourceA,
    env: { ENV1: "", ENV2: "" },
    ingresses: [ingressA],
    egresses: [egressA]
};
```

## Ingress
An Ingress represents an entry point into a Cell or Component. An `ingress` is defined as follows:
```
cellery:Ingress ingress = {
    name: string,
    port: (string) "containerPort:targetPort",
    context: string,
    definitions?: [{Definition}*],
};
```

`port` is defined as `"containerPort:targetPort"`. Ex: `"8080:80"`

A sample ingress instance would be as follows;
```
cellery:Ingress ingressA = {
    name: "foo",
    port: "8080:80",
    context: "foo",
    definitions: [
        {
            path: "*",
            method: "GET,POST,PUT,DELETE"
        }
    ]
};
```

#### Definitions
`definitions` represents the resources of a particular Ingress.
```
cellery:Definition definition = {
    path: string,
    method: string
}
```

## Egress
An Egress represents an external communication from a Cell or Component. An `egress` is defined as follows:
```
cellery:Egress egress = {
    parent: string "component name"
    ingress: Ingress,
    envVar: string,
    resiliency?: Resiliency,
};
```

A sample egress instance would be as follows;
```
cellery:Egress egressA = {
    parent: componentB.name,
    ingress: componentB.ingresses[0],
    envVar: "ENV1",
    resiliency: {
        retryConfig: {
            interval: 100,
            count: 10,
            backOffFactor: 0.5,
            maxWaitInterval: 20000
        },
        failoverConfig {
            endpoiint: "bar.com/failover",
            timeOut: 60
        }
    }
};
```

#### Resiliency
A `Resiliency` represents the resiliency configurations of an egress. A `Resiliency` is defined as follows;
```
cellery:Resiliency resiliency = {
    (RetryConfig|FailoverConfig)+ 
}
```

###### RetryConfig
A `RetryConfig` represents retry configurations for an egress endpoint. A `RetryConfig` is defined as follows;
```
cellery:RetryConfig retryConfig = {
    interval: int,
    count: int,
    backOffFactor: float,
    maxWaitInterval: int
}
```

###### FailoverConfig
A `FailoverConfig` represents fail over configurations for an egress endpoint. A `FailoverConfig` is defined as follows;
```
cellery:FailoverConfig failoverConfig = {
    endpoint: string,
    timeOut: int
}
```

## API
An API represents a defined set of functions and procedures that the services of the Components inside a Cell exposes 
as resources (i.e ingresses). An `API` is defined as follows;
```
cellery:API api = {
    name: string
    parent: string "component name"
    context: string|Ingress,
    global: boolean,
    definitions?: ingress|Definition[]
}
```

A sample API instance would be as follows;
```
cellery:API api = {
    name: "FooAPI,
    parent: componentA.name,
    ingress: componentA.ingresses[0],
    context: "/foo",
    global: true,
    definitions: [
        {
            path: "*",
            method: "GET,POST,PUT,DELETE"
        }
    ]
};
```

##
**Next:** [Modularity & Versioning](modularity.md)