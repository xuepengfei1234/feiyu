#region 工作流:提交、保存历史、添加待办
/// <summary>
/// 工作流:提交、保存历史、添加待办
/// </summary>
/// <param name="db">数据库上下文</param>
/// <param name="wfWorkEntity">工作流运转所需要的信息</param>
/// <returns>0 失败，1 正常，2 审批结束</returns>
public int StartProcess(PmsSystemDbContext db, WfWorkEntity wfWorkEntity)
{
// 业务编号
string appId = wfWorkEntity.shipId + "#" + wfWorkEntity.workId;

// 获得流程定义
Wfprocess wfprocess = db.Set<Wfprocess>().Where(q => q.ProcessGUID.ToUpper() == wfWorkEntity.processGuid.ToUpper() && q.IsDelete == 0).OrderByDescending(e => e.Ver).FirstOrDefault();

// 获得版本号
int? ver = wfprocess.Ver;

if (wfprocess == null)
{
throw new Exception("流程定义为空,请检查");
}
else
{
// 获得流程实例
Wfprocessinstance wfprocessinstanceInfo = db.Set<Wfprocessinstance>().SingleOrDefault(q => q.AppId == appId && q.IsProcessCompleted == 0 && q.IsDelete == 0);

if (wfprocessinstanceInfo != null)
{
return 0;
}
// 添加流程实例
Wfprocessinstance wfProcessInstance = new Wfprocessinstance();
// 流程实例ID
string ProcessInstanceID = Guid.NewGuid().ToString();
// 流程实例
wfProcessInstance.ProcessInstanceID = ProcessInstanceID;
// 流程定义编号
wfProcessInstance.ProcessGUID = wfprocess.ProcessGUID;
// 流程定义名称
wfProcessInstance.ProcessName = wfprocess.ProcessName;
// 业务编号
wfProcessInstance.AppId = appId;
// 是否完成 0 未完成
wfProcessInstance.IsProcessCompleted = WfStatus.ProcessNoCompleted;
// 创建时间
wfProcessInstance.CreatedDateTime = DateTime.Now;
// 船舶编号
wfProcessInstance.ShipId = wfWorkEntity.shipId;
// 获得工作流版本号
wfProcessInstance.Ver = ver;
// 创建人
wfProcessInstance.CreatedByUserName = wfWorkEntity.userName;
db.Set<Wfprocessinstance>().Add(wfProcessInstance);

// 添加活动实例
Wfactivityinstance wfActivityInstance = new Wfactivityinstance();
// 活动实例实ID
wfActivityInstance.ActivityInstanceID = Guid.NewGuid().ToString();
// 流程实例ID
wfActivityInstance.ProcessInstanceID = ProcessInstanceID;
// 业务ID
wfActivityInstance.AppId = appId;
// 流程定义ID
wfActivityInstance.ProcessGUID = wfprocess.ProcessGUID;
// 活动节点
wfActivityInstance.ActivityGUID = WfgroupExtend.WfFlowFiringtGuid;
// 活动名称
wfActivityInstance.ActivityName = wfWorkEntity.userName + (isEn == "cn" ? "提交" : "Submit");
// 是否完成
wfActivityInstance.IsActivityCompleted = WfStatus.ActivityCompleted;
// 创建人
wfActivityInstance.CreatedByUserName = wfWorkEntity.userName;
// 船舶编号
wfActivityInstance.ShipId = wfWorkEntity.shipId;
// 创建时间
wfActivityInstance.CreatedDateTime = DateTime.Now;
db.Set<Wfactivityinstance>().Add(wfActivityInstance);

// 添加日志
Wfold wfold = new Wfold();
// 获得日志最大编号
string wfoldId = Guid.NewGuid().ToString();
// 业务编号
wfold.Pid = appId;
// 流程定义编号
wfold.ProcessGUID = wfWorkEntity.processGuid;
// 执行人
wfold.Approver = wfWorkEntity.userName;
// 日期
wfold.ApproverDate = DateTime.Now;
// 流程名称
wfold.ActivityName = (isEn == "cn" ? "提交" : "Submit");
// 备注
wfold.Remark = (isEn == "cn" ? "提交" : "Submit");
// 意见
wfold.Memo = wfWorkEntity.approve_memo;
// 操作人
wfold.UserName = wfWorkEntity.userId;
// 编号
wfold.Id = wfoldId;
// 提交状态
wfold.WfState = 1;
// 删除标识
wfold.IsDelete = 0;
// 船舶编号
wfold.ShipId = wfWorkEntity.shipId;
// 流程实例
wfold.ProcessInstanceID = ProcessInstanceID;
// 添加审批历史
db.Set<Wfold>().Add(wfold);

// 该流程所有信息
List<Wfgroup> listWfGroup = db.Set<Wfgroup>().Where(q => q.ProcessGuid.ToUpper() == wfWorkEntity.processGuid.ToUpper() && q.IsDelete == 0 && q.Ver == ver).ToList();

// 获得第一步工作流程定义
Wfgroup group = new Wfgroup();

// 没有指定节点，按原先流程执行
if (string.IsNullOrEmpty(wfWorkEntity.SpecifiedNode))
{
group = db.Set<Wfgroup>().SingleOrDefault(q => q.ProcessGuid.ToUpper() == wfWorkEntity.processGuid.ToUpper() && q.ActivityGUID.ToUpper() == WfgroupExtend.WfFlowStartGuid.ToUpper() && q.Ver == ver && q.IsDelete == 0);

// 当前角色
string strPerofrmBy = group.GroupId;
// 是否活动实例是否直接结束 0：加待办 1：不加待办
int isActivityCompleted = 0;
// 下一步角色不为空进入
while (!string.IsNullOrEmpty(strPerofrmBy))
{
// 获得该流程当前审批步骤
Wfgroup wfGroup = db.Set<Wfgroup>().FirstOrDefault(q => q.ProcessGuid.ToUpper() == group.ProcessGuid.ToUpper() && q.GroupId.Contains(strPerofrmBy) && q.Ver == ver);

if (wfWorkEntity.listRole != null)
{
// 是否有这个角色,有当前角色,自动通过
if (wfWorkEntity.listRole.Count(e => e == strPerofrmBy) > 0)
{
isActivityCompleted = 1;
}
else
{
isActivityCompleted = 0;
}
}
else
{
isActivityCompleted = 0;
}
strPerofrmBy = wfGroup.NextGroupId;
// 会签/正常 待办
JointlySign(db, group.Branch, wfWorkEntity, wfProcessInstance, group, isActivityCompleted);

if (isActivityCompleted == 1)
{
if (wfWorkEntity.listRole.Count(e => e == strPerofrmBy) <= 0)
{
if ("NoNode" == strPerofrmBy)
{
return 2;
}
else
{
// 获得该流程当前审批步骤
Wfgroup newWfGroup = db.Set<Wfgroup>().FirstOrDefault(q => q.ProcessGuid.ToUpper() == group.ProcessGuid.ToUpper() && q.GroupId.Contains(strPerofrmBy) && q.Ver == ver);
if (newWfGroup != null)
{
// 会签/正常 待办
JointlySign(db, group.Branch, wfWorkEntity, wfProcessInstance, newWfGroup, 0);
}
return 1;
}
}
}

if (isActivityCompleted == 0)
{
return 1;
}
else if (isActivityCompleted == 1)
{
if ("NoNode" == strPerofrmBy)
{
return 2;
}
}
}
}
// 指定节点
else
{
group = db.Set<Wfgroup>().SingleOrDefault(q => q.ProcessGuid.ToUpper() == wfWorkEntity.processGuid.ToUpper() && q.ActivityGUID.ToUpper() == wfWorkEntity.SpecifiedNode.ToUpper() && q.IsDelete == 0 && q.Ver == ver);
// 添加活动实例
AddWfActivityInstance(db, wfProcessInstance, wfWorkEntity.shipId, wfWorkEntity.workId, wfWorkEntity.userName, group, 0);
// 添加代办
wfWorkEntity.activityGuid = group.ActivityGUID;
wfWorkEntity.groupID = group.GroupId;
// 添加代办
SaveFlowProcess(db, wfWorkEntity);
}
}
return 1;
}
#endregion
