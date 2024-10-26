
# Modais em Blazor

Este documento fornece uma visão geral da implementação de modais em uma aplicação Blazor, especificamente para criar e editar categorias.

## Componentes de Modal

### Modal de Criação de Categoria

```razor
<Modal T="CategoriaAddDTO"
       IsOpen="isCreateCategoryModalOpen"
       OnConfirm="@OnCreateCategoryConfirm"
       OnCancel="@OnCreateCategoryCancel"
       ActionType="creating/categoria"
       SelectedDTO="object">
</Modal>
```

### Modal de Edição de Categoria

```razor
<Modal T="CategoriaUpdateDTO"
       IsOpen="isEditCategoryModalOpen"
       OnConfirm="@OnEditCategoryConfirm"
       OnCancel="@OnEditCategoryCancel"
       ActionType="editing/categoria"
       SelectedDTO="CategoriaGetDTO"
       Selected="selectedCategory">
</Modal>
```

## Template de Modal

### Parâmetros

```razor
@typeparam T
@typeparam SelectedDTO

@code {
    [Parameter] public bool IsOpen { get; set; }
    [Parameter] public EventCallback<T?> OnConfirm { get; set; }
    [Parameter] public EventCallback OnCancel { get; set; }
    [Parameter] public string ActionType { get; set; }
    [Parameter] public SelectedDTO? Selected { get; set; }
    public bool IsLoading = false;

    private async Task ConfirmAsync(T value)
    {
        IsLoading = true;

        if (value != null)
        {
            await OnConfirm.InvokeAsync(value);
        } 
        else
        {
            await OnConfirm.InvokeAsync();
        }

        IsOpen = false;
        IsLoading = false;
    }

    private async Task CancelAsync()
    {
        await OnCancel.InvokeAsync();
        IsOpen = false;
    }

    private RenderFragment GetForm()
    {
        return ActionType switch
        {
            // Alerta
            "alerting/solicitacao" => (builder) =>
            {
                builder.OpenComponent<RequestedUserAlert<T>>(0);
                builder.AddAttribute(1, "OnCancel", EventCallback.Factory.Create(this, CancelAsync));
                builder.CloseComponent();
            },
            // Criando
            "creating/categoria" => (builder) =>
            {
                builder.OpenComponent<CategoryModal<T, SelectedDTO>>(0);
                builder.AddAttribute(1, "ActionType", "creating/categoria");
                builder.AddAttribute(2, "OnSubmit", EventCallback.Factory.Create<T>(this, ConfirmAsync));
                builder.AddAttribute(3, "OnCancel", EventCallback.Factory.Create(this, CancelAsync));
                builder.AddAttribute(4, "IsLoading", IsLoading);
                builder.CloseComponent();
            },
            // Editando
            "editing/categoria" => (builder) =>
            {
                builder.OpenComponent<CategoryModal<T, SelectedDTO>>(0);
                builder.AddAttribute(1, "ActionType", "editing/categoria");
                builder.AddAttribute(2, "OnSubmit", EventCallback.Factory.Create<T>(this, ConfirmAsync));
                builder.AddAttribute(3, "OnCancel", EventCallback.Factory.Create(this, CancelAsync));
                builder.AddAttribute(4, "SelectedDTO", Selected);
                builder.AddAttribute(5, "IsLoading", IsLoading);
                builder.CloseComponent();
            },
            // Deletando
            "deleting/categoria" => (builder) =>
            {
                builder.OpenComponent<DeleteModal<T>>(0);
                builder.AddAttribute(1, "ActionType", "deleting/categoria");
                builder.AddAttribute(2, "OnSubmit", EventCallback.Factory.Create<T>(this, ConfirmAsync));
                builder.AddAttribute(3, "OnCancel", EventCallback.Factory.Create(this, CancelAsync));
                builder.AddAttribute(4, "IsLoading", IsLoading);
                builder.CloseComponent();
            },
            // Padrão
            _ => (builder) =>
            {
                builder.AddContent(0, "Tipo de ação desconhecido.");
            }
        };
    }
}
```

```razor
<div class="modal-overlay @(IsOpen ? "show" : "")">
    <div class="modal-content @(IsOpen ? "show" : "")" style="@(ActionType == "view/produto" ? "padding: 0 !important;" : "")">
        @GetForm()
    </div>
</div>
```

## Template de Categoria Modal

```razor
@typeparam T
@typeparam SelectedDTOParam

@code {
    [Parameter] public string ActionType { get; set; } 
    [Parameter] public EventCallback<T> OnSubmit { get; set; }
    [Parameter] public EventCallback OnCancel { get; set; }
    [Parameter] public SelectedDTOParam? SelectedDTO { get; set; }
    [Parameter] public bool IsLoading { get; set; }

    private string Title => ActionType == "creating/categoria" ? "Criar Categoria" : "Editar Categoria";
    private string categoryName;
    private string selectedColor = "#272A2D";

    protected override void OnParametersSet()
    {
        if (ActionType == "editing/categoria" && SelectedDTO is CategoriaGetDTO categoria)
        {
            categoryName = categoria.Nome;
            selectedColor = categoria.BackgroundColor;
        }
    }

    private async Task SubmitAsync()
    {
        if (ActionType == "creating/categoria")
        {
            await sendCreate();
            return;
        }

        if (ActionType == "editing/categoria")
        {
            await sendEdit();
            return;
        }
    }

    private async Task CancelAsync()
    {
        await OnCancel.InvokeAsync();
    }

    private async Task sendCreate()
    {
        CategoriaAddDTO data = new CategoriaAddDTO
        {
            Nome = categoryName,
            BackgroundColor = selectedColor
        };

        await OnSubmit.InvokeAsync((T)(object)data);
    }

    private async Task sendEdit()
    {
        CategoriaUpdateDTO data = new CategoriaUpdateDTO
        {
            Nome = categoryName,
            BackgroundColor = selectedColor
        };

        await OnSubmit.InvokeAsync((T)(object)data);
    }
}
```

```razor
<div>...</div>
