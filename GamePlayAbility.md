# GameplayAbility 具体流程是怎样？？
- TryActivateAbilitiesByTag
  - TryActivateAbility

    - FindAbilitySpecFromHandle
    - CanActivateAbility

    - InternalTryActivateAbility
      - CallActivateAbility
        - PreActivate
        - ActivateAbility
      - SetCurrentActivationInfo
      - MarkAbilitySpecDirty

# GameplayAbility
- AbilitymanagerConponent

- GamePlayAbility
  - CancelAbilitiesWithTags
    - 打断任何其他技能
  - BlockAbilitesWithTag
    - 规定了“当A释放时，B类技能无法释放”。
  - ActivationOwnedTags
    - 释放前置标签列表
  - ActivationBlockedTags
    - 具有这些标签时，无法释放这个技能
  - TryActivateAbilityByClass
    -

- GamePlayEffect
  - GameplayEffect是不写事件的，它仅仅是一个数据储存结构！
  - DurationPolicy选择HasDuration表示有时长限制
  - Scalable Float表示时长的相乘系数
  - 天赋加成列表GrantedTags
  - Added栏写的是Buff，那么Removed栏写的是Debuff
  - CD也是用GameplayEffect来实现的

- GameplayAttributes

- GameplayCues

- FGameplayTag

- UGameplayTask


- FActiveGameplayEffectHandle
- FGameplayEffectContextHandle
- FGameplayEffectSpecHandle
- FGameplayAbilitySpecHandle
- FGameplayEffectQuery
- FAggregator
- FOnGameplayEffectAppliedDelegate
- FAbilityTargetDataSetDelegate
- FGameplayAbilityRepAnimMontage
