# AnimInstance
-  USkeletaMeshComponent ::InitAnim
  - InitializeAnimScriptInstance
    - AnimScriptInstance = NewObject<UAnimInstance>(this, AnimClass);
    - AnimScriptInstance->InitializeAnimation();
      - NativeInitializeAnimation
      - BlueprintInitializeAnimation

- TickAnimation
  - AnimInstance->UpdateAnimation()
    - PreUpdateAnimation
    - UpdateMontage
    - NativeUpdateAnimation
    - BlueprintUpdateAnimation
    - PostUpdateAnimation

- PostAnimEvaluation
  - 
