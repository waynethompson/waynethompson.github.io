---
layout: post
title:  "Generating Typescript Enums correctly with NSwag from Swagger docs created with Swashbuckle."
date:   2020-03-27 15:51:00 +1000
categories: [typescript, swagger, development]
tags: [development, nswag]
permalink: /blog/how-to-remove-x-powered-by-header-in-net-core/
---

We have been generating typescript classes and data clients for our angular application with NSwag by by pointing it at our AspNetCore api swagger documentation which was generated using swashbuckle.

When generating typescript enums this caused a problem that the enum option names were generated at numbers.


    // How it was by default 
    // This is not what we want
    exports enum SomeEnum {
            _1 = 1
            _2  = 2
        }

For obvious reasons we wanted the option names to be valid names indicating what they referred to. Something like:

     // This is how we want it
     exports enum SomeEnum {
        OptionOne = 1
        AnotherCoolOption  = 2
    }

After some searching we realised this was a limitation of NSwag generating from swashbuckle. If we used NSwag server side to generate the swagger docs this wouldn't happen.

What was required was a swashbuckle filter in the pipeline to add additional metadata to the enum schema.
   

     public class XEnumNamesSchemaFilter : ISchemaFilter
        {
            public void Apply(OpenApiSchema schema, SchemaFilterContext context)
            {
                var type = context.Type;
                if (type.IsEnum)
                {
                    // Add enum type information once
                    if (schema.Extensions.ContainsKey("x-enumNames")) return;
    
                    var valuesArr = new OpenApiArray();
                    valuesArr.AddRange(Enum.GetNames(context.Type)
                                                    .Select(value => new OpenApiString(value)));
    
                    schema.Extensions.Add(
                        "x-enumNames",
                        valuesArr
                    );
                }
            }
        }

To register it the following needed to be added to startup.cs

    services.AddSwaggerGen(c =>
    {
        c.SchemaFilter<XEnumNamesSchemaFilter>();
    });