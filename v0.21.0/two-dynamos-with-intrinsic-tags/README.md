# AWS SAM CLI Bug Report for v0.20.0

Seems to be an issue when two or more `AWS::DynamoDB::Table` resources (perhaps other resources as well?) use conditional intrinsic functions employing `!Ref AWS::NoValue`.

## Steps to reproduce
1. Install aws sam cli version 0.21.0
1. `git clone https://github.com/danludwig/aws-sam-cli-bug-reports.git`
1. `cd ./v0.21.0/two-dynamos-with-intrinsic-tags`
1. `sam build`

## Severity

Moderate. Results in having to modify buildspec.yaml files to force aws sam cli version 0.19.0 or lower before building the project.

## Details

This example was built from scratch with minor modifications to a new `sam init` project.

The bug does not seem to surface when the stack template only contains a single DynamoDB resource.

Commenting out the sections which use the `!If` / `!Ref AWS::NoValue` blocks results in a successful build with both DynamoDB resources. For example, the following will result in a successul build:

```

  DynamoDbTableOne:
    Type: AWS::DynamoDB::Table
    Properties:
      # Tags:
      #   - Key: Project
      #     Value: !Ref ProjectName
      #   - !If
      #     - CreateBackups
      #     - Key: data:backup
      #       Value: ""
      #     - !Ref AWS::NoValue
      AttributeDefinitions:
        - AttributeName: id
          AttributeType: S
      KeySchema:
        - KeyType: HASH
          AttributeName: id
      BillingMode: PAY_PER_REQUEST

  DynamoDbTableTwo:
    Type: AWS::DynamoDB::Table
    Properties:
      # Tags:
      #   - Key: Project
      #     Value: !Ref ProjectName
      #   - !If
      #     - CreateBackups
      #     - Key: data:backup
      #       Value: ""
      #     - !Ref AWS::NoValue
      AttributeDefinitions:
        - AttributeName: id
          AttributeType: S
      KeySchema:
        - KeyType: HASH
          AttributeName: id
      BillingMode: PAY_PER_REQUEST

```

Build error stack trace:

```
$ sam build
Unable to process properties of DynamoDbTableOne.AWS::DynamoDB::Table
Traceback (most recent call last):
  File "runpy.py", line 193, in _run_module_as_main
  File "runpy.py", line 85, in _run_code
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\samcli\__main__.py", line 12, in <module>
    cli(prog_name="sam")
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\click\core.py", line 764, in __call__
    return self.main(*args, **kwargs)
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\click\core.py", line 717, in main
    rv = self.invoke(ctx)
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\click\core.py", line 1137, in invoke
    return _process_result(sub_ctx.command.invoke(sub_ctx))
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\click\core.py", line 956, in invoke
    return ctx.invoke(self.callback, **ctx.params)
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\click\core.py", line 555, in invoke
    return callback(*args, **kwargs)
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\click\decorators.py", line 64, in new_func
    return ctx.invoke(f, obj, *args, **kwargs)
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\click\core.py", line 555, in invoke
    return callback(*args, **kwargs)
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\samcli\lib\telemetry\metrics.py", line 94, in wrapped
    raise exception  # pylint: disable=raising-bad-type
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\samcli\lib\telemetry\metrics.py", line 65, in wrapped
    return_value = func(*args, **kwargs)
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\samcli\commands\build\command.py", line 105, in cli
    skip_pull_image, parameter_overrides, mode)  # pragma: no cover
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\samcli\commands\build\command.py", line 138, in do_cli
    mode=mode) as ctx:
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\samcli\commands\build\build_context.py", line 65, in __enter__
    self._function_provider = SamFunctionProvider(self._template_dict, self._parameter_overrides)
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\samcli\commands\local\lib\sam_function_provider.py", line 46, in __init__
    self.template_dict = SamBaseProvider.get_template(template_dict, parameter_overrides)
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\samcli\commands\local\lib\sam_base_provider.py", line 53, in get_template
    template_dict = resolver.resolve_template(ignore_errors=True)
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\samcli\lib\intrinsic_resolver\intrinsic_property_resolver.py", line 240, in resolve_template
    processed_template["Resources"] = self.resolve_attribute(self._resources, ignore_errors)
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\samcli\lib\intrinsic_resolver\intrinsic_property_resolver.py", line 265, in resolve_attribute
    processed_resource = self.intrinsic_property_resolver(val, parent_function=processed_key)
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\samcli\lib\intrinsic_resolver\intrinsic_property_resolver.py", line 220, in intrinsic_property_resolver
    sanitized_val = self.intrinsic_property_resolver(val, parent_function=parent_function)
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\samcli\lib\intrinsic_resolver\intrinsic_property_resolver.py", line 220, in intrinsic_property_resolver
    sanitized_val = self.intrinsic_property_resolver(val, parent_function=parent_function)
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\samcli\lib\intrinsic_resolver\intrinsic_property_resolver.py", line 199, in intrinsic_property_resolver
    return [self.intrinsic_property_resolver(item) for item in intrinsic]
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\samcli\lib\intrinsic_resolver\intrinsic_property_resolver.py", line 199, in <listcomp>
    return [self.intrinsic_property_resolver(item) for item in intrinsic]
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\samcli\lib\intrinsic_resolver\intrinsic_property_resolver.py", line 213, in intrinsic_property_resolver
    return self.conditional_key_function_map.get(key)(intrinsic_value)
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\samcli\lib\intrinsic_resolver\intrinsic_property_resolver.py", line 753, in handle_fn_if
    intrinsic_value, parent_function=IntrinsicResolver.FN_IF
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\samcli\lib\intrinsic_resolver\intrinsic_property_resolver.py", line 199, in intrinsic_property_resolver
    return [self.intrinsic_property_resolver(item) for item in intrinsic]
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\samcli\lib\intrinsic_resolver\intrinsic_property_resolver.py", line 199, in <listcomp>
    return [self.intrinsic_property_resolver(item) for item in intrinsic]
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\samcli\lib\intrinsic_resolver\intrinsic_property_resolver.py", line 210, in intrinsic_property_resolver
    return self.intrinsic_key_function_map.get(key)(intrinsic_value)
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\samcli\lib\intrinsic_resolver\intrinsic_property_resolver.py", line 659, in handle_fn_ref
    return self._symbol_resolver.resolve_symbols(arguments, IntrinsicResolver.REF)
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\samcli\lib\intrinsic_resolver\intrinsics_symbol_table.py", line 217, in resolve_symbols
    translated = self.get_translation(logical_id, resource_attribute)
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\samcli\lib\intrinsic_resolver\intrinsics_symbol_table.py", line 323, in get_translation
    return logical_id_item.get(resource_attributes)
AttributeError: 'NoneType' object has no attribute 'get'
```