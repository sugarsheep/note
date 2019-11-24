

# 系统经常蓝屏，代码CRITICAL_PROCESS_DIED

#【方法A】 卸载可能导致蓝屏的软件、停止非核心的程序运作（包括第三方杀毒、优化软体）： 

-360AntiHijac（360防护软体） 
-pcdsrvc_x64.sys（Dell SupportAssist 软体） 
-TesSafe.sys(腾讯相关软件） 
-WeGameDriver（腾讯游戏） 

1. 卸载设备中的第三方杀毒、管家、优化软件 
2. 同时按【Windows 徽标键+R】，输入 【msconfig】，按回车(Enter) 
3. 点击 【服务】>【隐藏所有 Microsoft 服务】>【全部禁用】 
4. 启动【任务管理器】，点击 【启动】 选项卡，将所有启动项都禁用 
5. 重启设备 

======================================== 
#【方法B】 使用《sfc /scannow》和《Dism》自动扫描和修补系统档案： 

1. 右键点选桌面左下角Windows图标，选择【Microsoft Powershell(管理员)】 
2. 逐一输入以下指令： 
Dism /Online /Cleanup-Image /CheckHealth，按回车 
Dism /Online /Cleanup-Image /ScanHealth，按回车 
Dism /Online /Cleanup-Image /RestoreHealth，按回车 
sfc /scannow，按回车 
3. 重启电脑 
4. 如果出现”有一些文件无法修复“的回报，重复步骤1-3几次