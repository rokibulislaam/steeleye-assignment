## Explain what the simple List component does

The `WrappedListComponent` component is memoized with React's `memo` performance optimization API and named `List` in the codebase. The component accepts a prop `items` which should be a type of array of objects in the shape: `Array<{text: string}>`. The `WrappedListComponent` renders a list of `SingleListItem` mapping over `items` prop, and the component also keeps a track of which `SingleListItem` has been selected in `selectedIndex` state. The List component basically renders a list, and changes the background color of list item from green to red when clicked on.

## What problems / warnings are there with code?

- **Bug-1**: onClickHandler function is immediately invoked in `onClick` in `li` tag in `WrappedSingleListItem` component.

  Before:

  ```jsx
  <li onClick={onClickHandler(index)}>...</li>
  ```

  After:

  ```jsx
  <li onClick={() => onClickHandler(index)}>...</li>
  ```

- **Bug-2**: Array has been destructured in wrong order in `WrappedListComponent` component.

  Before:

  ```jsx
  const [setSelectedIndex, selectedIndex] = useState();
  ```

  After:

  ```jsx
  const [selectedIndex, setSelectedIndex] = useState(null);
  ```

- **Improvement-1**: There's no need to use useEffect here, since items dependency is not changing throughout its lifecycle. And even though if items prop changes, the component will re-render anyway.

  Before:

  ```jsx
  useEffect(() => {
    setSelectedIndex(null);
  }, [items]);
  ```

  After:

  ```jsx
  // useEffect(() => {
  //   setSelectedIndex(null);
  // }, [items]);
  ```

- **Improvement-2**: key prop has been added so that react can keep track of the list component and handle its internal performance optimization.

  Before:

  ```jsx
  <SingleListItem>...</SingleListItem>
  ```

  AFter:

  ```jsx
  <SingleListItem key={index}>...</SingleListItem>
  ```

- **Bug-3**: Previously only index was provided which doesn't make sense, since we are tracking the index of the listItem which is selected in selectedIndex state. So we should compare the index of the listItem and selectedIndex state, which would return boolean value.

  Before:

  ```jsx
    <SingleListItem isSelected={selectedIndex}>...<SingleListItem/>
  ```

  After:

  ```jsx
  <SingleListItem isSelected={selectedIndex === index}>...<SingleListItem/>
  ```

- **Bug-4**:

  Before:

  ```jsx
  WrappedListComponent.propTypes = {
    items: PropTypes.array(
      PropTypes.shapeOf({
        text: PropTypes.string.isRequired,
      })
    ),
  };
  ```

  After:

  ```jsx
   WrappedListComponent.propTypes = {
    items: PropTypes.arrayOf(
      PropTypes.shape({
        text: PropTypes.string.isRequired,
      })
    );
  };
  ```
## Please fix, optimize, and/or modify the component as much as you think is necessary.
The codebase has been modified and optimized. Code after fixing & optimizing:

```jsx
import React, { useState, /*useEffect,*/ memo } from 'react';

/**
 * There's no need to import useEffect, later explained why it has been removed
 */

import PropTypes from 'prop-types';

// Single List Item
const WrappedSingleListItem = ({
  index,
  isSelected,
  onClickHandler,
  text,
}) => {
  return (
    <li
      style={{ backgroundColor: isSelected ? 'green' : 'red' }}
      /**
       * Before:
       * onClick={onClickHandler(index)}
       * 
       * After:
       */
      onClick={() => onClickHandler(index)}
    /**
     * It should be function, previously the function was immediately
     * invoked. Now it won't be immediately invoked, rather will be
     * invoked when onclick event ocurred
     */
    >
      {text}
    </li>
  );
};

WrappedSingleListItem.propTypes = {
  index: PropTypes.number,
  isSelected: PropTypes.bool,
  onClickHandler: PropTypes.func.isRequired,
  text: PropTypes.string.isRequired,
};

const SingleListItem = memo(WrappedSingleListItem);

// List Component
const WrappedListComponent = ({
  items,
}) => {

  /**
   * Before:
   *   const [setSelectedIndex, selectedIndex] = useState();
   * After: 
   */
  const [selectedIndex, setSelectedIndex] = useState(null);

  /**
   * Before: 
   */

  // useEffect(() => {
  //   setSelectedIndex(null);
  // }, [items]);

  /**
   * There's no need to use useEffect here, since items dependency
   * is not changing throughout its lifecycle. And even though if items
   * prop changes, the component will re-render anyway 
   */

  const handleClick = index => {
    setSelectedIndex(index);
  };

  return (
    <ul style={{ textAlign: 'left' }}>
      {items?.map((item, index) => (
        <SingleListItem
          /**
           * Before: no key prop
           * After: 
           */

          key={index}
          /**
           * key prop has been added so that react can keep track
           * of the list component and handle its internal 
           * performance optimization
           */

          onClickHandler={() => handleClick(index)}
          text={item.text}
          index={index}
          /**
           * Before:
           *  isSelected={selectedIndex}
           * 
           * After:
           */
          isSelected={selectedIndex === index}
          /**
           * Previously only index was provided which doesn't 
           * make sense, since we are tracking the index of the
           * listItem which is selected in selectedIndex state.
           * So we should compare the index of the listItem and 
           * selectedIndex state, which would return boolean value.
           */
        />
      ))}
    </ul>
  )
};



WrappedListComponent.propTypes = {
  /**
   * Before: 
   *   items: PropTypes.array(PropTypes.shapeOf({
   * 
   * After:
   */
  items: PropTypes.arrayOf(PropTypes.shape({
    text: PropTypes.string.isRequired,
  }))
}

WrappedListComponent.defaultProps = {
  items: null,
};

const List = memo(WrappedListComponent);

// export default List;


export default function App() {
  return (
    <div>
      <List items={(() => {
        const arr = []
        for (let i = 0; i < 10; i++) {
          arr.push({ text: `This is line ${i + 1}` })
        }
        return arr;
      })()} />
    </div>
  )
}

```