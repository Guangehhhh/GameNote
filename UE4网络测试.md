# 网络log
- GameMode Login函数
- PostLogin函数
- ForceNetUpdate

- 看PossessBy的区别
- SetAutonomousProxy
- ForcePropertyCompare
- 	CopyRemoteRoleFrom(GetDefault<APawn>());
- 	NetworkPredictionInterface->ResetPredictionData_Server();
if (!IsLocalPlayerController())
		{
			GetPawn()->Restart();
		}
    ClientRestart(GetPawn());
ChangeState( NAME_Playing );
UpdateNavigationComponents();
