# Step Function

- Server-less function orchestrator that makes it easy to sequence AWS lambda functions & 
    multiple AWS services into business-critical applications.
- AWS services can contains lambda, SQS, SNS and Fargate.
- Error handling, retries can be done.


### Task State - It is used to accomplish unit of work.

```
    "HelloWorld": {
        "Type": "Task",
        "Resource": "arn:aws:lambda:us-east-2....",
        "Next": "AfterHelloWorldState",
        "Comment": "Run the HelloWorld Lambda function"
    }
```

### Wait State - It is used to pause the state execution.
- You can wait till one year at max.

```
    "wait_ten_seconds": {
        "Type": "Wait",
        "Seconds": 10,
        "Next": "NextState"
    }
```

### Wait State - It is used to execute two states in parallel.
- If one of the parallel state fails, entire state files.

```
    {
        "Comment": "Parallel Example.",
        "StartAt": "LookupCustomerInfo",
        "States": {
            "LookupCustomerInfo": {
                "Type": "Parallel",
                "End": true,
                "Branches": [
                    {
                        "StartAt": "LookupAddress",
                        "States": {
                            "LookupAddress": {
                                "Type": "Task",
                                "Resource": "arn:aws:lambda:us-east-2....",
                                "End": true,
                            }
                        }
                    },
                    {
                        "StartAt": "LookupPhone",
                        "States": {
                            "LookupAddress": {
                                "Type": "Task",
                                "Resource": "arn:aws:lambda:us-east-2....",
                                "End": true,
                            }
                        }
                    }
                ]
            }
        }
    }
```

### Choice State - It is used to select the next state based on the conditions

```
    "DisptachEvent": {
        "Type":"Choice",
        "Choices": [
            {
                "Not": {
                    "Variable": "$.type",
                    "StringEquals": "Private"
                },
                "Next": "Public"
            },
            {
                "And": [
                    {
                        "Variable": "$.value",
                        "IsPresent": true
                    },
                    {
                        "Variable": "$.value",
                        "IsNumeric": true
                    },
                    {
                        "Variable": "$.value",
                        "NumericGreaterThanEquals": 20
                    },
                    {
                        "Variable": "$.value",
                        "NumericLessThan": 30
                    },
                    "Next": "ValueInTwenties"
                ]
            }
        ],
        "Deafult": "RecordEvent"
    }
```

### Pass State - It is used to append or add more attributes to the input
- If $ is not used in the result path then entire input will be replaced with Result

```
Input: 
{
    "georefOf": "Home"
}

Pass State:
"No-op": {
    Type: "Pass",
    "Result": {
        "x-datum": 0.381018,
        "y-datum": 622.22699
    }
    "ResultPath": "$.coords",
    "Next": "End"
}

Output:
{
    "georefOf": "Home",
    "coords": {
        "x-datum": 0.381018,
        "y-datum": 622.22699
    }
}
```

### Succeed State - It is used to represent the success state.

```
"SuccessState": {
    "Type": "Succeed"
}
```

### Fail State - It is used to represent the fail state.

```
"FailState": {
    "Type": "Fail",
    "Cause": "Invalid response.",
    "Error": "ErrorA"
}
```

### Map State - It is used to Execute multiple inputs parallely.

```
Input:
{
    "ship-date":"2016-07-06",
    "detail": {
        "delivery-partner": "UQS",
        "shipped": [
            {"prod": "R31", "dest-code": 9511, "quantity": 1344},
            {"prod": "S39", "dest-code": 9511, "quantity": 40}
        ]
    }
}

Map State:

"Validate-All": {
    "Type": "Map",
    "InputPath": "$.detail",
    "ItemsPath": "$.shipped",
    "Maxconcurrency": 0,
    "Iterator": {
        "StartAt": "Validate",
        "States": {
            "Validate": {
                "Type": "Task",
                "Resource": "",
                "End": true
            }
        }
    },
    "ResultPath": "$.detail.shipped",
    "End": true
}
```

