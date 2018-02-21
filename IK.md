## 5/24 IK系统研究

* IK
  * 首先写一个从Socket到地面的射线碰撞  
  * 返回一个旋转值和浮点数
  * 然后Interp插值旋转值
  * Interp 插值浮点数
  * Hip Zoffset函数
  * 本地到组件空间
* 时间回复功能
  * 物体每秒计算tick保存到transform 数组里；
  * 在按键后读取数组删除读取的数 循环设置transform

  * 在工具->选项->项目和解决方案->VC++目录
  里面把包含文件和库文件路径添加
  *  project->property->linker->general->Additional library directories 填写lib路径
  project->property->linker->input 填写lib名称
