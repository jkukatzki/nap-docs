Scene Management {#scene}
=======================
*	[Overview](@ref scene_overview)
*	[Example](@ref scene_setup)
*	[Resources vs Instances](@ref resources_instances)
*	[Components](@ref component_ov)	
	*	[Creating Components](@ref creating_components)
		*	[The Resource](@ref component_resource)
		*	[The Instance](@ref component_instance)
*	[Creating Links](@ref scene_linking)
	*	[To Components](@ref component_link)
			*	[Array of Components] (@ref multiple_component_link)
	*	[To Entities](@ref entity_link)
			*	[Array of Entities](@ref multiple_entity_link)
*	[Instance Properties](@ref instance_props_scene)


Overview {#scene_overview}
=======================

Modern applications can grow considerably in size when it comes to the amount of data they have to manage and the complex logic they need to support. NAP uses a powerful [entity component system](https://en.wikipedia.org/wiki/Entity_component_system) to aid the development process. This system allows you to identify and organize all the essential parts of your application. What separates entities and components from regular resources is that they allow you to group content together in a meaningful way and are dynamic. They are created to perform a specific task at runtime of your application. For example: render your model at a specific location, position the camera or play back audio. All the individual parts in the system can reference and talk to each other in a generic fashion. 

Listed below are the most important objects and their roles within the system:

- Every application has one or more [scenes](@ref nap::Scene)
- A scene contains a hierarchy of [entities](@ref nap::Entity)
- Entites hold [components](@ref nap::Component)
- Components add functionality to an entity, ie: define behaviour
- Entities and components can link to each other
- Entities are [instantiable](@ref nap::EntityInstance), ie: you can create multiple versions based on the same template
- [Components](@ref nap::ComponentInstance) are updated every frame

Example {#scene_setup}
=======================

Consider this 'Bicycle' example:

```
Scene:
	Entity: "Bicycle"
		Component: "BikePosition"
		Component: "BikeInput"
		Entity: "Frame"
			Entity: "Wheel"
				Component: "WheelPosition"
			Entity: "Wheel"
				Component: "WheelPosition"
 ```

In this example we have a single scene. This scene contains exactly one `Bicycle`. Every bicycle has a frame and 2 wheels. The bicycle, frame and wheels are entities. The names of these entities are very descriptive: they allow you to identify the different parts of the bike. Entities can form a hierchy: the `Frame` is a child of the `Bicycle` and the wheels are children of the `Frame`. 

The `BikeInput` component moves the bike in a direction when it receives input (by updating the `BikeTransform`). Every entity in the scene has a position and that position is always relative to the position of its parent. People that work with 3D applications should recognize this pattern. 

In this example the bike has two wheels, one for the front and one for the back. They are both the same 'wheel' but placed differently. Other parts that could vary between wheels are the wheel color or size. What this means is that the same wheel is 'instantiated' twice, but with different properties. You can change these properties for every wheel individually but the template for both wheels is the same 'Wheel' entity. 

You use [Napkin](@ref napkin) to create this entity structure for you. When saved, the end result looks something like this in JSON:

```
{
	"Type" : "nap::Scene",
	"mID": "Scene",
	"Entities" : 
	[
		{
			"Entity" : "Bicycle"
		}
	]
},
{
	"Type" : "nap::Entity",
	"mID": "Wheel"
},
{
	"Type" : "nap::Entity",
	"mID": "Frame",
	"Children" :
	[
		"Wheel",
		"Wheel"
	]
},
{
	"Type" : "nap::Entity",
	"mID": "Bicycle",
	"Children" :
	[
		"Frame"
	]
}
```

To make things a bit easier to read we have removed the components.

Resources vs Instances {#resources_instances}
=======================

We briefly touched upon the difference between resources and instances in the example above. Resources are 'simple' data containers that can be authored in Napkin. Resources are rather static and often remain in their original state. An instance, on the other hand, is never part of a data file. An instance only exists inside the running application and is created by NAP. An [Entity](@ref nap::Entity) is such a resource. It has a [runtime counterpart](@ref nap::EntityInstance) that is updated by NAP every frame. 

The position of the bike (in the example above) changes as it moves through the world. The initial position of the bike is declared in Napkin but the runtime position changes each frame. When there are multiple bikes in the scene, each bike has its own position. As a programmer you want to change the position of each bike programmatically, ie: set it based on a set of conditions. When you do that you modify the run-time state of a bike, not the resource that was used to create 'an instance of' the bike.

To summarize:
- Resources contain static, shared, read-only data
- Instances contain runtime-varying data and can be updated each frame

Both the Entity and Component have a resource and instance counterpart. NAP omits the resource part of the class name for readability:

- An [Entity](@ref nap::Entity) becomes an [EntityInstance](@ref nap::EntityInstance)
- A [Component](@ref nap::Component) becomes a [ComponentInstance](@ref nap::ComponentInstance)

Similar to regular resources, an entity (resource) is created in the `Resources` panel in Napkin. The entity is instantiated by NAP when it is added to the `Scene`. You can add the same entity multiple times to the same scene, with varying component properties. For every entity that is added to the scene NAP creates an instance. At run-time, in your application, you work with instances of entities and components.

Components {#component_ov}
=======================

A scene contains entities. An entity has components. Scenes and entities do not execute any behavior by themselves. They allow you to group and organize your objects. Components are used to add functionality to an entity, ie: defining its behavior. It is the component that receives an [init()](@ref nap::ComponentInstance::init) and [update()](@ref nap::ComponentInstance::update) call. Any programmable behavior is therefore executed in the component.

NAP offers a number of components off the shelf such as the [TransformComponent](@ref nap::TransformComponent) and [RenderableMeshComponent](@ref nap::RenderableMeshComponent). These can be used to build hierarchies of visual objects. There are however many more components that ship with NAP. Most modules expose their own components, including: input, OSC, midi and audio components.

You probably want to create new components for specific tasks. The video modulation demo uses two custom components. Both components are only available to the videomodulation application. One of these components allows the user to select a shape from a selection of three-dimensional shapes. The component makes sure that every shape in the list can be rendered to screen and offers an interface to select the one to draw.

Creating Components {#creating_components}
-----------------------

A component is a regular resource that you author in Napkin. Therefore, everything you know about [resources](@ref resources) also applies to components. But the component has, as mentioned before, a run time counterpart in the form of an instance. To make a new component you have to create (and register) both sides: the resource and instance. The video modulation demo makes use of custom components. Refer to that code for a good working example. 

### The Resource {#component_resource} ###

To create the resource part of a new component derive your class from [Component](@ref nap::Component):

~~~~~~~~~~~~~~~{.cpp}
class NAPAPI PerspCameraComponent : public Component
{
	RTTI_ENABLE(Component)
	DECLARE_COMPONENT(PerspCameraComponent, PerspCameraComponentInstance)

	virtual void getDependentComponents(std::vector<rtti::TypeInfo>& components) const override 
	{ 
		components.push_back(RTTI_OF(TransformComponent)); 
	}

public:
	float mFieldOfView = 50.0f;				// Property: Camera Field Of View
};
~~~~~~~~~~~~~~~

.. and register it together with the properties you need:

~~~~~~~~~~~~~~~{.cpp}
RTTI_BEGIN_CLASS(PerspCameraComponent)
	RTTI_PROPERTY("mFieldOfView", &PerspCameraComponent::mFieldOfView,  nap::rtti::EPropertyMetaData::Default)
RTTI_END_CLASS
~~~~~~~~~~~~~~~

Here we create a perspective camera with a field of view property. Some concepts are familiar, others are new:

- We derive from [Component](@ref nap::Component) instead of [Resource](@ref nap::Resource)
- The `DECLARE_COMPONENT` macro tells the system which instance of this component to create
- [getDependentComponents()](@ref nap::Component::getDependentComponents) tells the system that this component depends on a transform 

If your component depends on another component you can `hint` at it. In the example above the perspective camera needs a transform to position itself in the world. NAP will make sure that if a transform is available it is initialized before the camera is initialized. Subsequently: initialization of the camera fails if the transform is not available. We can extend the scene with an entity that holds both the new component and a transform component:

```
{
    "Type" : "nap::Scene",
    "mID": "Scene", 
    "Entities" : 
    [
        {
            "Entity" : "Bike",
            "Entity" : "CameraEntity"
        }
    ]
},
{
    "Type" : "nap::Entity",
    "mID": "CameraEntity",
    "Components" : 
    [
        {
            "Type" : "nap::PerspCameraComponent",
            "mID" : " PerspCameraComponent",
            "FieldOfView" : 90
        },
        {
            "Type" : "nap::TransformComponent"
        }
    ]
}
```

### The Instance {#component_instance} ###

When calling [loadFile()](@ref nap::ResourceManager::loadFile()) the perspective camera component is created as part of the 'CameraEntity'. NAP will now attempt to create the run time counterpart (instance) of the `PerspCameraComponent`: a `PerspCameraComponentInstance`. To create the instance part of a new component derive your class from [ComponentInstance](@ref nap::ComponentInstance):

~~~~~~~~~~~~~~~{.cpp}
class NAPAPI PerspCameraComponentInstance : public ComponentInstance
{
	RTTI_ENABLE(ComponentInstance)
public:
	PerspCameraComponentInstance(EntityInstance& entity, Component& resource);

	/**
	 * Copy over the field of view property value
	 * and find the transform component
	 */
	virtual bool init(utility::ErrorState& errorState) override;
};
~~~~~~~~~~~~~~~

This part of the new component needs to be registered in the .cpp as well. Because every instance is created at run-time and never read from file you don't have to register any properties. You only register the class. The init pattern and the error handling is exactly the same as with regular resources. A small difference is the fact that this object does not contain a default constructor. The constructor of a component instance receives:

- the entity this component belongs to.
- the resource that created this instance. 

To make sure this object can be created by NAP we have to tell the system what constructor to use. When registering the instance part you explicitly tell the system that there is no default constructor available and add one explicitly:

~~~~~~~~~~~~~~~{.cpp}
RTTI_BEGIN_CLASS_NO_DEFAULT_CONSTRUCTOR(nap::PerspCameraComponentInstance)
	RTTI_CONSTRUCTOR(nap::EntityInstance&, nap::Component&)
RTTI_END_CLASS
~~~~~~~~~~~~~~~

The instance part of the component is created when the file is loaded and the system encounters a `PerspCameraComponent` resource. The instance is constructed using the registered constructor and is given the component (resource) that created it and the entity (instance) it belongs to. When the instance is initialized you know that the transform is available and everything up to that point went well. You can now safely locate the transform component and use it to (for example) compute the camera position in world space. 

The [update()](@ref nap::ComponentInstance::update) function can be overridden to add per-frame functionality to your instance. The system calls the `update` function together with a time stamp for you. The camera instance in the example above doesn't need it but other components do, for example: a component that blends two lines over time.

Creating Links {#scene_linking}
=======================

You already know how to [create a link](@ref pointing) to a resource. But you can't use this type of link to point to components and entities. After all: there could be multiple instances of the same entity or component in your scene. When you want to point to a component you use the [ComponentPtr](@ref nap::ComponentPtr). When you want to point to an entity you use the [EntityPtr](@ref nap::EntityPtr). Pointing to components and entities is almost the same as pointing to stand-alone resources with one exception: you can use absolute or relative paths. It is recommended to use our editor (Napkin) to create links between objects in the scene for you. 

To Components {#component_link}
-----------------------

The [ComponentPtr](@ref nap::ComponentPtr) allows you to point (create a link) to another component in the scene. It works almost the same as [pointing to resources](@ref pointing) but needs a bit of extra information to work. Let's look at the resource part of a component first:

~~~~~~~~~~~~~~~{.cpp}
class LineBlendComponent : public Component
{
	RTTI_ENABLE(Component)
	DECLARE_COMPONENT(LineBlendComponent, LineBlendComponentInstance)

public:
	// property: Link to selection component one
	ComponentPtr<LineSelectionComponent> mSelectionComponentOne;

	// property: Link to selection component two
	ComponentPtr<LineSelectionComponent> mSelectionComponentTwo;

	// property: link to the mesh to store the blend result
	ResourcePtr<nap::PolyLine> mTarget;
};
~~~~~~~~~~~~~~~

This component blends two lines based on a blend value. The end result is stored in 'mTarget'. The blend target is a link to a regular resource ([ResourcePtr](@ref nap::rtti::ObjectPtr)). Both input lines are extracted from a different component that live under the same entity, in this case a line selection component. Input one and two are therefore links to a different component and are required by the blend component to perform the blend operation.

The registration of this part of the component in the .cpp file should look familiar. Links to components are registered as regular properties:

~~~~~~~~~~~~~~~{.cpp}
RTTI_BEGIN_CLASS(nap::LineBlendComponent)
	RTTI_PROPERTY("SelectionOne",	&nap::LineBlendComponent::mSelectionComponentOne,	nap::rtti::EPropertyMetaData::Required)
	RTTI_PROPERTY("SelectionTwo",	&nap::LineBlendComponent::mSelectionComponentTwo,	nap::rtti::EPropertyMetaData::Required)
	RTTI_PROPERTY("Target",			&nap::LineBlendComponent::mTarget,					nap::rtti::EPropertyMetaData::Required)
RTTI_END_CLASS
~~~~~~~~~~~~~~~  

When NAP encounters this component it tries to resolve both links for you. Both selection components are created before the blend component is created and set as links on the instance part of the blend component. But the system needs to know where to store the result of this operation. The instance part of the component exposes two members that (after serialization) link to the correct runtime version of the newly created selection components:

~~~~~~~~~~~~~~~{.cpp}
class LineBlendComponentInstance : public ComponentInstance
{
	RTTI_ENABLE(ComponentInstance)
public:
	LineBlendComponentInstance(EntityInstance& entity, Component& resource) :
		ComponentInstance(entity, resource)			{}

		ComponentInstancePtr<LineSelectionComponent> mSelectorOne = { this, &LineBlendComponent::mSelectionComponentOne };
		ComponentInstancePtr<LineSelectionComponent> mSelectorTwo = { this, &LineBlendComponent::mSelectionComponentTwo };
}
~~~~~~~~~~~~~~~

The [ComponentInstancePtr](@ref nap::ComponentInstancePtr) is the runtime counterpart of the [ComponentPtr](@ref nap::ComponentPtr). Both links are set before `init()` is called. If the system can't resolve the link NAP will cancel the load operation and return an error message. The template argument is the resource part of the component this member links to. The construction arguments are always 'itself' and a reference to the original link. You don't have to specify anything else. The system will populate both members for you after construction! The registration of the instance part of the component in the .cpp is always the same:

~~~~~~~~~~~~~~~{.cpp}
RTTI_BEGIN_CLASS_NO_DEFAULT_CONSTRUCTOR(nap::LineBlendComponentInstance)
	RTTI_CONSTRUCTOR(nap::EntityInstance&, nap::Component&)
RTTI_END_CLASS
~~~~~~~~~~~~~~~

You can now create the links in Napkin, with as a possible result in JSON:
```
{
	"Type" : "nap::Entity",
	"mID": "LaserEntity",
	"Components" : 
	[
		{
			"Type" : "nap::LineSelectionComponent",
			"mID" : "LineSelectorOne",
			"Lines" :
			[
				"LineFromFile1",
				"LineFromFile2"
			]
		},
		{
			"Type" : "nap::LineSelectionComponent",
			"mID" : "LineSelectorTwo",
			"Lines" :
			[
				"LineFromFile4",
				"LineFromFile5"
			]
		},
		{
			"Type" : "nap::LineBlendComponent",
			"mID" : "LineBlender",
			"SelectionComponentOne" : "./LineSelectorOne",
			"SelectionComponentTwo" : "./LineSelectorTwo",
			"Target" : "LaserLineMesh1"
		}
	]
}
```

Notice how we use a relative path? We tell the system to look for the selection component on the same entity. You can also walk further up or down the tree to create links to other components. More on that in a later section.

### Array of Components {#multiple_component_link} ###

Instead of linking to a single component it is possible to link to multiple components at once. The components are grouped together in an array and all components in that array share the same base class. The only difference (from a single link) is that the definition and declaration of the link is a vector, just as you would normally use a vector in C++. Based on the previous example we can group both links into an array: 

~~~~~~~~~~~~~~~{.cpp}
class LineBlendComponent : public Component
{
	RTTI_ENABLE(Component)
	DECLARE_COMPONENT(LineBlendComponent, LineBlendComponentInstance)

public:
	// property: Link to selection component one
	std::vector<ComponentPtr<LineSelectionComponent>> mComponents;

	// property: link to the mesh to store the blend result
	ResourcePtr<nap::PolyLine> mTarget;
};
~~~~~~~~~~~~~~~

The registration in the .cpp file should look familiar. Instead of defining both links separately we define only the array as a property:

~~~~~~~~~~~~~~~{.cpp}
RTTI_BEGIN_CLASS(nap::LineBlendComponent)
	RTTI_PROPERTY("Components",			&nap::LineBlendComponent::mComponents,	nap::rtti::EPropertyMetaData::Required)
	RTTI_PROPERTY("Target",				&nap::LineBlendComponent::mTarget,		nap::rtti::EPropertyMetaData::Required)
RTTI_END_CLASS
~~~~~~~~~~~~~~~  

The instance part of the 'LineBlendComponent' also requires an array, but instead of holding a list of resources it holds a list of instances. The system automatically resolves the links for you. After initialization, you are able to access the individual components immediately in your application:

~~~~~~~~~~~~~~~{.cpp}
class LineBlendComponentInstance : public ComponentInstance
{
	RTTI_ENABLE(ComponentInstance)
public:
	LineBlendComponentInstance(EntityInstance& entity, Component& resource) :
		ComponentInstance(entity, resource)			{}

		std::vector<ComponentInstancePtr<LineSelectionComponent>> mComponents = initComponentInstancePtr(this, &LineBlendComponent::mComponents);
}
~~~~~~~~~~~~~~~

After that you can add the links in Napkin, with as a possible result in JSON:
```
{
	"Type" : "nap::Entity",
	"mID": "LaserEntity",
	"Components" : 
	[
		...

		{
			"Type" : "nap::LineBlendComponent",
			"mID" : "LineBlender",
			"Target" : "LaserLineMesh1",
			"Components" : 
			[
				"./LineSelectorOne",
				"./LineSelectorTwo"
			]
		}
	]
}
```

To Entities {#entity_link}
-----------------------

The [EntityPtr](@ref nap::EntityPtr) allows you to point (create a link) to another entity in the scene. They work exactly the same as a [component pointer](@ref component_link) and are created in a similar way. Notice that the template argument is not necessary because there is only one type of entity:

~~~~~~~~~~~~~~~{.cpp}
class NAPAPI CameraController : public Component
{
	RTTI_ENABLE(Component)
	DECLARE_COMPONENT(CameraController, CameraControllerInstance)
public:
	nap::EntityPtr	mLookAtTarget;			///< Entity to look at
}
~~~~~~~~~~~~~~~

Links to entities are also a property of the component. The registration of the resource part of this component is therefore rather straight forward:

~~~~~~~~~~~~~~~{.cpp}
RTTI_BEGIN_CLASS(nap::CameraController)
	RTTI_PROPERTY("LookAtTarget",	&nap::CameraController::mLookAtTarget,	nap::rtti::EPropertyMetaData::Required)
RTTI_END_CLASS
~~~~~~~~~~~~~~~

The instance also has a link to the lookat target that is resolved after load. For more information take a look at [creating component links](@ref component_link):

~~~~~~~~~~~~~~~{.cpp}
class NAPAPI CameraControllerInstance : public ComponentInstance
{
	RTTI_ENABLE(ComponentInstance)
public:
	CameraControllerInstance(EntityInstance& entity, Component& resource);
	...
	EntityInstancePtr mLookAtTarget = { this, &CameraController::mLookAtTarget };	// The resolved runtime lookat target
}
~~~~~~~~~~~~~~~

You don't have to register any properties in the .cpp file regarding the instance part of the component:

~~~~~~~~~~~~~~~{.cpp}
RTTI_BEGIN_CLASS_NO_DEFAULT_CONSTRUCTOR(nap::CameraControllerInstance)
	RTTI_CONSTRUCTOR(nap::EntityInstance&, nap::Component&)
RTTI_END_CLASS
~~~~~~~~~~~~~~~

You can now edit the link in Napkin. This works the same as authoring links to components. See example above.

### Array of Entities {#multiple_entity_link} ###

Instead of linking to a single entity it is possible to link to multiple entities at once. The entities are grouped together in an array. The declaration of this array is the same as declaring an [array of component pointers](@ref multiple_component_link), but uses ['initEntityInstancePtr()'](@ref nap::initEntityInstancePtr) instad of ['initComponentInstancePtr()'](@ref nap::initComponentInstancePtr). To declare an array of entity pointers as a property of a component use a standard C++ vector: ['std::vector<nap::EntityPtr>'](@ref nap::EntityPtr).

Instance Properties {#instance_props_scene}
=======================

Instance properties override the default property value of a component. They are applied on top of the default (shared) value. This allows you to `spawn` the same entity with slightly different settings each time. For example: a different color, start position, acceleration speed etc. 

Napkin can [create](@ref in_prop_override) component property overrides for you.