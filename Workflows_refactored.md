# Модель данных

`WorkflowNode` - это сущность с необязательными параметрами, которые задаются при создании. 
Имеет смысл изменить поведение таким образом, чтобы у каждого конкретного узла была своя роль.
При таком рефакторе мы сможем на стороне модели данных (шаблонов) определять обязательные и не обязательные поля для каждого узла. (Это первое преимущество, что обнаружил, лежит на поверхности)

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