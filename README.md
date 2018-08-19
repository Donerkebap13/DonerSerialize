![Doner Serializer](https://i.imgur.com/DOJNofX.png)

[![Release version](https://img.shields.io/badge/release-v1.0.0-blue.svg)](https://github.com/Donerkebap13/DonerSerializer/releases/tag/1.0.0) [![Build Status](https://travis-ci.org/Donerkebap13/DonerSerializer.svg?branch=master)](https://travis-ci.org/Donerkebap13/DonerSerializer) [![Build status](https://ci.appveyor.com/api/projects/status/tvfolb6nui3eflyq/branch/master?svg=true)](https://ci.appveyor.com/project/Donerkebap13/donerserializer/branch/master)

## A C++14 header-only library to serialize your class data to JSON

**DonerSerializer** is a C++14 header-only library that provides you a simple interface to serialize/deserialize your class data in a few lines of code.

Internally it uses: 
- [DonerReflection](https://github.com/Donerkebap13/DonerReflection)
- [RapidJson](https://github.com/Tencent/rapidjson)
## Supported types
**Built-in types**
- ``std::int32_t``
- ``std::uint32_t``
- ``std::int64_t``
- ``std::uint64_t``
- ``float``
- ``double``
- ``bool``

**Std containers**
- ``std::string``
- ``std::vector``
- ``std::list``
- ``std::map``
- ``std::unordered_map``

**[User-defined Types](#how-to-serialize-your-custom-classes)**


## Downloading

You can acquire stable releases [here](https://github.com/Donerkebap13/DonerSerializer/releases).

Alternatively, you can check out the current development version with:

```
git clone https://github.com/Donerkebap13/DonerSerializer.git
```
## Contact

You can contact me directly via [email](mailto:donerkebap13@gmail.com).
Also, if you have any suggestion or you find any bug, please don't hesitate to [create a new Issue](https://github.com/Donerkebap13/DonerReflection/issues).

If you decide to start using **DonerSerializer** in your project, I'll be glad to hear about it and post it here in the main page as an example!
## How to use it
**DonerSerializer** uses **[DonerReflection](https://github.com/Donerkebap13/DonerReflection)** macros to expose your class data.
```c++
namespace Foo
{
	struct Bar
	{
		int m_int;
		float m_float;
		const char* m_char;
	}
}
// IMPORTANT!!
// It is mandatory that this macro calls happen always outside of any namespace
DONER_DEFINE_REFLECTION_DATA(Foo::Bar,
	DONER_ADD_NAMED_VAR_INFO(m_int, "intFoo"),
	DONER_ADD_NAMED_VAR_INFO(m_float, "floatBar"),
	DONER_ADD_NAMED_VAR_INFO(m_char, "charMander")
)
```
As simple as that. Those macros will define a struct containing all the registered members info for that class. As the comment say, **this calls should be done outside of any namespace**. Otherwise it won't work.
Also, if you don't care about how each member is called in the json, you can use the following macro instead:
```c++
DONER_DEFINE_REFLECTION_DATA(Foo::Bar,
	DONER_ADD_VAR_INFO(m_int),
	DONER_ADD_VAR_INFO(m_float),
	DONER_ADD_VAR_INFO(m_char)
)
```
The name in this case will be the same as the variable name.

In order to access protected/private members, you need to use the following macro:
```c++
namespace Foo
{
	class Bar
	{
	DONER_DECLARE_OBJECT_AS_REFLECTABLE(Bar)
	private:
		int m_int;
		float m_float;
		const char* m_char;
	}
}
// IMPORTANT!!
// It is mandatory that this macro calls happen always outside of any namespace
DONER_DEFINE_REFLECTION_DATA(Foo::Bar,
	DONER_ADD_NAMED_VAR_INFO(m_int, "intFoo"),
	DONER_ADD_NAMED_VAR_INFO(m_float, "floatBar"),
	DONER_ADD_NAMED_VAR_INFO(m_char, "charMander")
)
```
By doing this you allow **DonerSerializer** to access private members information.
### Inheritance
**DonerSerializer** doesn't support inheritance per se. If you want to serialize class members inherited from its upper class, you need to do as follows:
```c++
class Foo
{
protected:
	int m_int;
}

class Bar : public Foo
{
DONER_DECLARE_OBJECT_AS_REFLECTABLE(Bar)
private:
	float m_float;
}

DONER_DEFINE_REFLECTION_DATA(Bar,
	DONER_ADD_VAR_INFO(m_int),
	DONER_ADD_VAR_INFO(m_float)
)
```
Even if the upper class have some reflection data defined, **this information is not transitive**, so you need to re-declare it for any children class.

## How to Serialize
You just need to use ``DonerSerializer::CJsonSerializer``
```c++
CFoo foo;
foo.m_int = 1337;

DonerSerializer::CJsonSerializer serializer;
serializer.Serialize(foo);
std::string result = serializer.GetJsonString(); // value is {"m_int": 1337}
```
You can also get the ``rapidjson::Document`` with all the contents:
```c++
rapidjson::Document& document = serializer.GetJsonDocument();
```
If you rather prefer to use your own ``rapidjson::Document``, you can use the static method ``Serialize``:
```c++
CFoo foo;
foo.m_int = 1337;
rapidjson::Document document;
// ...
// Any changes to document
// ...
DonerSerializer::CJsonSerializer::Serialize(foo, document);
```
## How to Deserialize
You just need to load the json and use the static method ``CJsonDeserializer::Deserialize``
```c++
CFoo foo;
DonerSerializer::CJsonDeserializer::Deserialize(foo, "{\"m_int\": 1337}");
// foo.m_int == 1337 
```
You can also specify a specific ``rapidjson::Value`` to deserialize data from:
```c++
rapidjson::Value value;
// ...
// Fill value with some data
// ...
CFoo foo;
DonerSerializer::CJsonDeserializer::Deserialize(foo, value);
```
## How to Serialize your custom classes
In order to serialize you own classes, you just need to inherit from ``DonerSerialization::ISerializable`` and to define the desired reflection data as [mentioned above](#how-to-use-it)
```c++
class Foo : DonerSerialization::ISerializable
{
DONER_DECLARE_OBJECT_AS_REFLECTABLE(Foo)
protected:
	int m_int;
}
DONER_DEFINE_REFLECTION_DATA(Foo,
	DONER_ADD_VAR_INFO(m_int)
)
// ...
class Bar : public Foo
{
DONER_DECLARE_OBJECT_AS_REFLECTABLE(Bar)
private:
	Foo m_foo;
}
DONER_DEFINE_REFLECTION_DATA(Bar,
	DONER_ADD_VAR_INFO(m_foo)
)
```
## How to Serialize Thirdparty types
A thirdparty type is a type defined in any external library, where you can't change the implementation of the types to make them usable by **DonerSerializer**. 

To achieve this, you can specialize ``CDeserializationResolver::CDeserializationResolverType<>`` and ``CSerializationResolver::CSerializationResolverType<>``. Here's an example on how to do it for ``SFML`` ``sf::Vector2f``:
```c++
namespace DonerSerializer
{
	template <>
	class CDeserializationResolver::CDeserializationResolverType<sf::Vector2f>
	{
	public:
		static void Apply(sf::Vector2f& value, const rapidjson::Value& att)
		{
			if (att.IsArray())
			{
				value = sf::Vector2f(att[0].GetFloat(), att[1].GetFloat());
			}
		}
	};
	
	template <>
	class CSerializationResolver::CSerializationResolverType<sf::Vector2f>
	{
	public:
		static void Apply(const char* name, const sf::Vector2f& value, rapidjson::Document& root)
		{
			rapidjson::Value array(rapidjson::kArrayType);
			CSerializationResolver::CSerializationResolverType<float>::SerializeToJsonArray(array, value.x, root.GetAllocator());
			CSerializationResolver::CSerializationResolverType<float>::SerializeToJsonArray(array, value.y, root.GetAllocator());
			root.AddMember(rapidjson::GenericStringRef<char>(name), array, root.GetAllocator());
		}

		static void SerializeToJsonArray(rapidjson::Value& root, const sf::Vector2f& value, rapidjson::Document::AllocatorType& allocator)
		{
			rapidjson::Value array(rapidjson::kArrayType);
			CSerializationResolver::CSerializationResolverType<float>::SerializeToJsonArray(array, value.x, allocator);
			CSerializationResolver::CSerializationResolverType<float>::SerializeToJsonArray(array, value.y, allocator);
			root.PushBack(array, allocator);
		}
	};
}
```