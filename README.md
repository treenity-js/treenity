# treenity

![logo](https://res.cloudinary.com/dxjccseos/image/upload/w_170,f_auto/treenity-logo)

[![Netlify Status](https://api.netlify.com/api/v1/badges/4bcf05dc-54c2-46da-9ca9-e3b4b59cdabe/deploy-status)](https://app.netlify.com/sites/treenity/deploys)

Treenity is a flexible, performant and unstyled component that is not dependent on any data structure, CSS or any UI library.

The component offer a set of hooks for managing the tree state and HOC's and an enhancer for managing sections inside each item.

In the repo you can explore few examples that covers some common use cases such as styling and windowing for performance boost.

Please use it wisely :)

### Table Of Contents

- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Data Structure](#data-structure)
- [Hooks](#hooks)
  - [useSelected](#useselected)
  - [useExpanded](#useexpanded)
  - [useLoading](#useloading)
- [getItemProps](#getitemprops)
  - [combineClickProps](#combineClickProps)
- [Sections](#sections)
  - [enhancedGetItemPropsWithSections](#enhancedGetItemPropsWithSections)
  - [HOC](#hoc)
- [Advanced Usage](#advanced-usage)

# Installation

```javascript
npm install treenity
```

```javascript
yarn add treenity
```

# Basic Usage

```javascript
import React, { memo } from 'react';
import { getItemProps, useExpanded, useSelected } from 'treenity';

const Item = memo(({ label, expanded, visible, getSelectedProps, getExpandedProps }) => (
  <>
    {visible && (
      <div {...getSelectedProps()}>
        <button {...getExpandedProps()}>{expanded ? '-' : '+'}</button>
        {label}
      </div>
    )}
  </>
));

const Tree = () => {
  const selectedProps = useSelected();
  const expandedProps = useExpanded();

  return (
    <>
      {data.map(item => (
        <Item key={item.id} {...getItemProps({ ...item, ...selectedProps, ...expandedProps })} />
      ))}
    </>
  );
};
```

# Data Structure

The idea behind this package is not to stick with any data structure, you can provide any structure you like. The motivation behind it that i was working on several projects that were using a tree but each one of them had a different structure, one add a "regular" structure and the second had a flat structure, and i wanted to have something generic that i can use for both.

In addition i didnt want to provide any UI items since each implementation is different, one was using `styled-components` and other `css modules`, so the responsability for building the UI components is on you!! The package provides some helpers and a lot of examples to help you do it right.

There are 2 parameters that are required for basic structuring of the tree:

| Name  | Type   | Desc                                                                                                 |
| ----- | ------ | ---------------------------------------------------------------------------------------------------- |
| id    | string | a _unique_ value of the item rendered                                                                |
| depth | number | the depth of the node, this parameter is important for setting up the state of each item in the tree |

For example:

```javascript
[
  { depth: 0, id: 'Treenity', label: 'Treenity' },
  { depth: 1, id: 'src', label: 'src' },
  { depth: 2, id: 'components', label: 'components' },
  { depth: 3, id: 'grid', label: 'grid' },
  { depth: 1, id: 'tests', label: 'tests' },
  { depth: 1, id: 'other', label: 'other' },
];
```

# Hooks

For managing the state of your tree component, the package provides the following hooks: `useSelected`, `useExpanded` and `useLoading`

Each hook expose some helpers functions that for managing the state.

## useSelected

```javascript
const { setSelected, isSelected } = useSelected();
```

#### setSelected

> function(id: string) | optional

Set the selected item by it's id. If passing no value then nothing will be selected

#### isSelected

> function(id: string): boolean | required

check if specific item is selected

## useExpanded

```javascript
const { setExpanded, isExpanded, isVisible } = useExpanded();
```

#### setExpanded

> function(id: string, isExpanded: boolean) | required

Set the expanded item by it's id.

#### isVisible

> function(id: string): boolean | required

check if specific item is visible

#### isExpanded

> function(id: string): boolean | required

check if specific item is expanded

Please note: If you will like to use this `isExpanded` and `isVisible` while managing your own custom logic, make sure to call both methods - start with `isVisible` method and end with `isExpanded` method - order is important!.

## useLoading

```javascript
const { setLoading, isLoading } = useLoading();
```

#### setLoading

> function(id: string, isLoading: boolean) | required

Set the loading item by it's id.

#### isLoading

> function(id: string): boolean | required

check if specific item is loaded

#getItemProps

> getItemProps(props: Object): Object

This method does all the magic by calculating each item state. The idea is to pass each of the state methods by using each of the hooks. The method triggers each of the hooks method automatically.

For example:

```javascript

const selectedProps = useSelected();
const expandedProps = useExpanded();
const loadingProps - useLoading();

<Item {...getItemProps({...item, ...selectedProps, ...expandedProps, ...loadingProps})}/>

```

#### Props returned by the method:

| Name        | Type     | Desc |
| ----------- | -------- | ---- |
| id          | string   |
| depth       | number   |      |
| visible     | boolean  |      |
| expanded    | boolean  |      |
| selected    | boolean  |      |
| loading     | boolean  |      |
| setSelected | function |      |
| setExpanded | function |      |
| setLoading  | function |      |

In addition there are extra helpers function you can use in your components for managing the click state

> getSelectedProps()

This method return an onClick event that will be automatically attached to the component and trigger the selection method

> getExpandedProps()

This method return an onClick event that will be automatically attached to the component and trigger the expanded method

> getKeyboardProps()

This method return an onKedown event that will be automatically attached to the component and will enable keyboard navigation on the tree items

```javascript

const Item = memo(({ label, visible, ...props }) => {
  const { getSelectedProps, getExpandedProps, getKeyboardProps } = props;

  return (
    <>
      {visible && (
        <div {...getKeyboardProps()} {...getSelectedProps()}}>
          <button {...getExpandedProps()}>+</button>
          {label}
        </div>
      )}
    </>
  );
}, areEqualDebug);

```

#### combineClickProps

In case you want to trigger both selected and expanded state together you can use the following helper function

```javascript
const { getExpandedProps, getSelectedProps } = props;
const onClick = useCallback(() => combineClickProps({ getExpandedProps, getSelectedProps })(), [getExpandedProps, getSelectedProps]);
```

## Sections

Sections is a way to add to each item header and footer since sometimes you will want to add extra information around your item which are not part of the data structure - for example you will wanna add a pagination part inside of each item.
The header will be displayed on top of the item and the footer at the bottom of the item - you can use both.

In order to view the section you will need to define for each item `itemHeader` and `itemFooter` params.

For example:

```javascript
  { id: 'Treenity', label: 'Treenity', depth: 0, more: true, itemHeader: true, itemFooter: true },
```

There are 2 ways of using sections - with the `enhancedGetItemPropsWithSections` method or by `withSections` HOC

### enhancedGetItemPropsWithSections

> enhancedGetItemPropsWithSections(props: Object): Array[Object] | Required: isLast: boolean

This method enhancing `getItemProps` and add to it the extra sections. The difference between both that `getItemProps` return an object representing the props of the item and `enhancedGetItemPropsWithSections` return and array of props that consists of the sections and the item.

In order for the logic to work correctly it is _required_ to pass a `isLast` boolean param since the method need to know when it reached the last item, and add all the leftover items to the last item.

The props returned for the sections items are:

| Name        | Type     | Desc                                                                  |
| ----------- | -------- | --------------------------------------------------------------------- |
| id          | string   | new ID based on the orignal one, with a suffic of `_HEADER` `_FOOTER` |
| origId      | string   | the original ID of the item, in order to keep a reference when needed |
| renderedId  | string   | the ID of the item that the section was attached to                   |
| footer      | boolean  | type of the section is footer                                         |
| header      | boolean  | type of the section is header                                         |
| depth       | number   |                                                                       |
| visible     | boolean  |                                                                       |
| expanded    | boolean  |                                                                       |
| selected    | boolean  |                                                                       |
| loading     | boolean  |                                                                       |
| setSelected | function |                                                                       |
| setExpanded | function |                                                                       |
| setLoading  | function |                                                                       |

```javascript

const getItemProps = enhancedGetItemPropsWithSections();

const Tree = () => {
  const stateProps = { ...useSelected(), ...useExpanded(), ...useLoading() };

  data.reduce((acc, { depth, ...item }, idx) => {
    const items = getItemProps({ depth, ...item, isLast: idx === data.length - 1, ...stateProps });

    items.forEach(item => {
      const { visible, ...props } = item;
      if (visible) acc.push({ visible, ...props });
    });

    return acc;
  }, []);
};

```

### withSections
```javascript
const ItemWithSection = withSections(Item, { Header, Footer });
```



#### need to talk about clearing the cache of lodash.memoize - if replacing the cache type - how do we handle ?
