# AWS SAM CLI Bug Report for v0.20.0

## Function using !Ref layer with !GetAtt to custom resources in environment variables.

Seems to be an issue when a function resource references a layer using `!Ref` against a parameter default and also contains environment variables whose values come from `!GetAtt` against custom cloudformation resources.

## Steps to reproduce
1. Install aws sam cli version 0.21.0
1. `git clone https://github.com/danludwig/aws-sam-cli-bug-reports.git`
1. `cd ./v0.21.0/lambda-layer-ref-with-default`
1. `sam build` or `sam build --use-container`
1. Examine error stack trace

## Severity

Moderate. Resulted in having to change from `!Ref LayerArn` to `!Sub ${LayerArn}`, which results in a different bug when running `sam local start-api`.

## Details

This example was built from scratch with minor modifications to a new `sam init` project.

The bug does not seem to surface when environemnt variable values only use `!Ref` against template parameters.

Commenting out environment variables which use `!GetAtt` against a custom resource cause the builds to succeed. For example, the following will result in a successful build:

```
      Environment:
        Variables:
          SAFE_ENVIRONMENT_VARIABLE_1:
            !Ref TemplateParameterOne
          # OFFENDING_ENVIRONMENT_VARIABLE:
          #   !GetAtt MyCustomResource.Parameter.Value
          SAFE_ENVIRONMENT_VARIABLE_2:
            !Ref MyCustomResource
```

Build error stack trace:

```
$ sam build
Unable to process properties of HelloWorldFunction.AWS::Serverless::Function
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
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\samcli\commands\local\lib\sam_function_provider.py", line 52, in __init__
    self.functions = self._extract_functions(self.resources)
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\samcli\commands\local\lib\sam_function_provider.py", line 100, in _extract_functions
    layers = SamFunctionProvider._parse_layer_info(resource_properties.get("Layers", []), resources)
  File "C:\Program Files\Amazon\AWSSAMCLI\runtime\lib\site-packages\samcli\commands\local\lib\sam_function_provider.py", line 263, in _parse_layer_info
    raise InvalidLayerReference()
samcli.commands.local.lib.exceptions.InvalidLayerReference: Layer References need to be of type 'AWS::Serverless::LayerVersion' or 'AWS::Lambda::LayerVersion'
```
