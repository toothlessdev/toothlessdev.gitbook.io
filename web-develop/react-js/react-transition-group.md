# React Transition Group 톺아보기



```tsx
import PropTypes from 'prop-types';
import React from 'react';
import TransitionGroupContext from './TransitionGroupContext';

import {
  getChildMapping,
  getInitialChildMapping,
  getNextChildMapping,
} from './utils/ChildMapping';

const values = Object.values || ((obj) => Object.keys(obj).map((k) => obj[k]));

const defaultProps = {
  component: 'div',
  childFactory: (child) => child,
};

export default class TransitionGroup extends React.Component {
  constructor(props, context) {
    super(props, context);

    const handleExited = this.handleExited.bind(this);

    this.state = {
      contextValue: { isMounting: true },
      handleExited,
      firstRender: true,
    };
  }

  componentDidMount() {
    this.mounted = true;
    this.setState({
      contextValue: { isMounting: false },
    });
  }

  componentWillUnmount() {
    this.mounted = false;
  }

  static getDerivedStateFromProps(
    nextProps,
    { children: prevChildMapping, handleExited, firstRender }
  ) {
    return {
      children: firstRender
        ? getInitialChildMapping(nextProps, handleExited)
        : getNextChildMapping(nextProps, prevChildMapping, handleExited),
      firstRender: false,
    };
  }

  handleExited(child, node) {
    let currentChildMapping = getChildMapping(this.props.children);

    if (child.key in currentChildMapping) return;

    if (child.props.onExited) {
      child.props.onExited(node);
    }

    if (this.mounted) {
      this.setState((state) => {
        let children = { ...state.children };

        delete children[child.key];
        return { children };
      });
    }
  }

  render() {
    const { component: Component, childFactory, ...props } = this.props;
    const { contextValue } = this.state;
    const children = values(this.state.children).map(childFactory);

    delete props.appear;
    delete props.enter;
    delete props.exit;

    if (Component === null) {
      return (
        <TransitionGroupContext.Provider value={contextValue}>
          {children}
        </TransitionGroupContext.Provider>
      );
    }
    return (
      <TransitionGroupContext.Provider value={contextValue}>
        <Component {...props}>{children}</Component>
      </TransitionGroupContext.Provider>
    );
  }
}
```
