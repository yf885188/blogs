# Level Load过程
- UWorld.InitWorld ： Levels.Add(PersistentLevel)
- UWorld.PersistentLevel : 在tick中更新，PersistentLevel = LevelCollections[ActiveLevelCollectionIndex]


# BlueprintGeneratedClass
- FLevelEditorActionCallbacks::OpenLevelBlueprint : 打开接口
- FBlueprintEditorModule::CreateBlueprintEditor ： 创建一个blue print editor
- FBlueprintEditor ： blue print editor
- FBlueprintEditor::Compile : 编译按钮响应
- FKismetEditorUtilities::CompileBlueprint
- FBlueprintCompilationManagerImpl::CompileSynchronouslyImpl
- FBlueprintCompilationManagerImpl::FlushCompilationQueueImpl

流程：
- STAGE I: GATHER
- STAGE II: FILTER
- STAGE III: SORT
- STAGE IV: SET TEMPORARY BLUEPRINT FLAGS
- STAGE V: VALIDATE
- STAGE VI: PURGE (LOAD ONLY)
- STAGE VII: DISCARD SKELETON CDO
- STAGE VIII: RECOMPILE SKELETON
- STAGE IX: RECONSTRUCT NODES, REPLACE DEPRECATED NODES (LOAD ONLY)
- STAGE X: CREATE REINSTANCER (DISCARD 'OLD' CLASS)
- STAGE XI: CREATE UPDATED CLASS HIERARCHY
- STAGE XII: COMPILE CLASS LAYOUT
- STAGE XIII: COMPILE CLASS FUNCTIONS
- STAGE XIV: REINSTANCE
- STAGE XV: POST CDO COMPILED 
- STAGE XVI: CLEAR TEMPORARY FLAGS

ULevel::OnLevelScriptBlueprintChanged ：关卡蓝图改变的回调

Reinstancer 创建：
```
CompilerData.Reinstancer = TSharedPtr<FBlueprintCompileReinstancer>(
					new FBlueprintCompileReinstancer(
						BP->GeneratedClass,
						CompileReinstancerFlags
					)
```

FBlueprintCompileReinstancer.DuplicatedClass 创建 :  FBlueprintCompileReinstancer::MoveCDOToNewClass


```
UClass* CopyOfOwnerClass = CastChecked<UClass>(StaticDuplicateObject(OwnerClass, GetTransientPackage(), ReinstanceName, ~RF_Transactional));
```


FLinkerLoad::SerializeExportMap
FLinkerLoad::PRIVATE_PatchNewObjectIntoExport
FLinkerLoad::FindExistingExport

FLinkerLoad::PopulateInstancingContext
FLinkerLoad::DeferExportCreation