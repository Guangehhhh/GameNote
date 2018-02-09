# Timer
- TimerManager

- SetTimer(TiemrHandle& ,UserClass*,<UserClass>::MeshodPtr  ,  Rate  )
  - InternalSetTimer(
                            TimerHandle,
                            FTimerUnifiedDelegate,
                            Rate,
                            bLoop,
                            FirstDelay)

- ClearTimer(TimerHandle& )
  - InternalClearTimer(TimerHandle& )
  - TimerHandle. Incalidate()

- PauseTimer(TimerHandle)
  - InternalPauseTimer(TimerDate,TimerIndex)

- AddOnScreenDebugMessage(
                                            Key,
                                            TimeToDisplay,
                                            DisplayColor,
                                            DubugMessage(FString&),
                                            bNewerOnTheTop,
                                            FVector2D&,
                                            )

# UE_LOG
DECLARE_LOG_CATEGORY_EXTERN(YourLog, Log, All);
DEFINE_LOG_CATEGORY
UE_LOG(LogTemp, Warning, TEXT("Your message"));



# Bug 2065
- Include"Engine.h"
