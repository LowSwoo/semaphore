# Модель данных

`WorkflowNode` - это сущность с необязательными параметрами, которые задаются при создании. 
Имеет смысл изменить поведение таким образом, чтобы у каждого конкретного узла была своя роль.

## А смысл?

### Обязательные поля
При таком рефакторе мы сможем на стороне модели данных (шаблонов) определять обязательные и не обязательные поля для каждого узла. (Это первое преимущество, что обнаружил, лежит на поверхности)

### Интерфейс
Сможем стандартизировать интерфейс узла. Это поможет понять и описать, а чего мы вообще от узла хотим?

### Создание задачи
Запуск таски сейчас происходит в одной функции с большим switch-case.

Прим:
```go
// pro_impl/services/server/workflow_svc.go — startWorkflowNode
case db.WorkflowNodeTaskKind:
	tpl, err := s.templateReceiver.GetTemplate(run.ProjectID, node.TemplateID)
	...
	var task db.Task
	if node.TaskParams != nil {
		task = node.TaskParams.CreateTask(node.TemplateID)
	} else {
		task = db.Task{TemplateID: node.TemplateID}
	}
	task.ProjectID = run.ProjectID
	task.BuildTaskID = buildTaskID
	task.WorkflowRunID = &run.ID
	task.WorkflowNodeID = &node.ID
	if run.Version != nil {
		task.Version = run.Version   // версия запуска перебивает версию из TaskParams
	}

	newTask, err := s.enqueuer.AddTask(task, userID, username, run.ProjectID, tpl.App.NeedTaskAlias())
```

После рефактора этот switch-case должен стать сильно проще. Это узел таски? Тогда запускаем соответствующую функцию для task узла. А в самой функции уже будут происходить нужные проверки, записи в бд и так далее.

## Примечания

Требуется продумать механизм работы с узлами, потому что текущий механизм будет сломан

## Было

```go
type WorkflowTemplate struct {
	ID           int
	ProjectID    int
	Name         string
	Description  *string
	StartVersion *string

	Nodes []WorkflowNode
	Edges []WorkflowEdge

	LastRun *WorkflowRun
}

type WorkflowNode struct {
	ID                 int
	WorkflowTemplateID int

	TemplateID      int 
	Kind            WorkflowNodeKind
	ConvergenceMode WorkflowConvergenceMode 

	ApprovalTimeout *int                    
	ApprovalMessage *string

	TaskParamsID *int                       
	TaskParams   *TaskParams                

	Note         *string                    
	DelaySeconds *int                       

	PositionX int                           
	PositionY int
}

type WorkflowEdge struct {
	ID                 int
	WorkflowTemplateID int
	SourceNodeID       int
	DestinationNodeID  int
	Condition          WorkflowEdgeCondition 
}
```

## Стало 
```go
type WorkflowTemplate struct {
	ID           int
	ProjectID    int
	Name         string
	Description  *string
	StartVersion *string

	Nodes []WorkflowNode  // Заменить на интерфейс, который будут реализовывать все узлы?
	Edges []WorkflowEdge

	LastRun *WorkflowRun
}

type WorkflowNode struct {
	ID                 int
	WorkflowTemplateID int

	PositionX int                           
	PositionY int
}

type WorkflowTaskNode struct {
    WorkflowNode

	TemplateID      int 
	Kind            WorkflowNodeKind
	ConvergenceMode WorkflowConvergenceMode 
	TaskParamsID    int                       
	TaskParams      TaskParams                
}

type WorkflowApprovalNode struct {
    WorkflowNode

	ApprovalTimeout int                    
	ApprovalMessage string

}

type WorkflowNoteNode struct {
    WorkflowNode

    Note string
}

type WorkflowDelayNode struct {
    WorkflowNode

	DelaySeconds int                       

}

type WorkflowEdge struct {
	ID                 int
	WorkflowTemplateID int
	SourceNodeID       int
	DestinationNodeID  int
	Condition          WorkflowEdgeCondition 
}
```