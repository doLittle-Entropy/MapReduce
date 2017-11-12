# Map / Reduce engine

The engine in the runtime will be interfaced by the different SDKs in a declarative manner.
Below is an example of how we see the C# version go.


```csharp
            Classify<GroupCreated>.AsAdd();
            Classify<GroupDeleted>.AsRemove();

            Classify<UserCreated>.AsAdd();
            Classify<UserDeleted>.AsRemove();

            Map
                .From<EmployeeHired>()
                .From<AddressSet>()
                .From<MobileNumberSet>()
                .From<EmployeeLeft>()
                .ReduceTo<TReadModel>(_ => {
                    _.WithId<UserCreated>(e => e.UserId)
                    _.WithPropertiesFrom<UserCreated>(readModel, source => {  // Should be possible to just specify eventType - no arguments, meaning it will map by convention (same name)
                        readModel.Username = source.Username;
                        readModel.Department = source.Department;
                        readModel.UserNumber = _r.Count()    // We need operation classifications (Add / Remove) to be able to do this
                    });
                    _.WithPropertiesFrom<EmailSet>(readModel, source => {
                        readModel.Email = source.Address;
                    });
                    _.WithPropertiesFrom<MobileNumberSet>(readModel, source => {
                        readModel.MobileNumber = source.Number;
                        readModel.Groups = _.WithCollectionOf<UserGroup>(_c => 
                        _c.WithpropertiesFrom<UserAddedToGroup>(readModel, source => {
                            readModel.GroupId = source.GroupId;
                        }));
                        _c.WithpropertiesFrom<GroupAdded>(readModel, source => {
                            readModel.Name = source.Name;
                        }));
                    });
                })
                GroupBy(_ => _.Department, _g => {
                    _g.ReduceToValue<UserCountPerDepartment>(_v => _v.Count())
                });
```