# jsx-directives

Директивы для JSX (React, Preact, др.) по аналогии с директивами [Vue](https://ru.vuejs.org/v2/guide/custom-directive.html) и [Angular](https://angular.io/guide/attribute-directives)

## Мотивация

Директивы позволяют по флагу добавить в компонент произвольное поведение. 
То есть мы выставляем в компоненте свойство и это имеет такой-же эффект, как если бы мы обернули его в HOC.
Директивы действуют глобально, в рамках приложения, и добавляют новый функционал во все компоненты. В отличие от HOC, их не нужно подключать для каждого компонента по отдельности.

<details>
<summary>Пример</summary>

Например, можно создать свойство для показа или скрытия элемента.
Тогда такой код:
    
```tsx
<div>
  {isVisible && <SomeElement />}
</div>
```
    
можно переписать так:
    
```tsx
<div>
  <SomeElement $visible={isVisible} />
</div>
```
    
В результате наш код стало проще читать и редактировать.
</details>

## Установка

Сначала устанавливаем пакет в приложение:
```bash
npm install jsx-directives
```

Далее нужно подключить директивы в главном файле приложения следующим образом:
```tsx
// Импортируем сам пакет директив
import { registerAllReact } from 'jsx-directives';

// Импортируем актуальные версии реакта и jsx-рантайма
import React from 'react'; // для TypeScript и других компиляторов
import * as rt from 'react/jsx-runtime'; // для production с компиляцией через Babel/CRA
import * as rtDEV from 'react/jsx-dev-runtime'; // для development с компиляцией через Babel/CRA

// Связываем директивы и актуальный реакт
registerAllReact(React, rt, rtDEV);
```

## Использование предустановленных директив

По умолчанию в пакете предустановлены следующие директивы:

- $show - принимает булево значение и добавляет элемент в DOM если значение истинно
- $hide - принимает булево значение и удаляет элемент из DOM если значение истинно
- $model - принимает объект и передаёт его свойства value и onChange как свойства компонента

Пример использования для React:
```tsx
import React, { FC, useCallback, useState } from 'react';

// Создаём вспомогательный хук, для удобства
function useModel<T>(defaultValue: T) {
  const [val, setVal] = useState(defaultValue);
  // useMemo не нужен, так как объект не дойдёт до реакта
  return  {
    value: val,
    onChange: useCallback((e: any) => {
      setVal(e.target.value);
    }, []),
  };
}

// Создаём сам компонент
export const SomeComponent: FC = () => {
  const model = useModel('');
  return (
    <div>
      <input $model={model} />
      <div>{model.value}</div>
      <div $hide={model.value === 'hide'}>Type "hide" and this message will be removed from DOM</div>
    </div>
  );
};
```

## Создание собственных директив

Пример создания директивы, модифицирующей пропсы:
```typescript
import { registerPropsDirective } from '@yandex-market/react-directives';

export type ModelType = {
    value: unknown;
    onChange: (...args: unknown[]) => void;
};

declare module 'react' {
    interface HTMLAttributes<T> extends DOMAttributes<T> {
        $model?: ModelType;
    }

    interface Attributes {
        $model?: ModelType;
    }
}

registerPropsDirective('$model', ({$model, ...props}: any) => ({
    value: $model.value,
    onChange: $model.onChange,
    ...props,
}));
```

Пример создания директивы, модифицирующей элемент:
```typescript
import { registerPropsDirective } from '@yandex-market/react-directives';

declare module 'react' {
    interface HTMLAttributes<T> extends DOMAttributes<T> {
        $show?: boolean;
    }

    interface Attributes {
        $show?: boolean;
    }
}

registerElementDirective('$show', (element: any, props: any) => {
    if (props && Boolean(props.$show)) {
        return null;
    } else {
        return element;
    }
});
```
