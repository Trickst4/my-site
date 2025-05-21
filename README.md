import _ from 'lodash';

const outputValue = (value) => {
  if (_.isObject(value)) {
    return '[complex value]';
  }
  if (typeof value === 'string') {
    return `'${value}'`;
  }
  return value;
};

const nodeFormatters = {
  added: (node, fullPath) => [
    `Property '${fullPath}' was added with value: ${outputValue(node.value)}`,
  ],
  removed: (node, fullPath) => [
    `Property '${fullPath}' was removed`,
  ],
  changed: (node, fullPath) => [
    `Property '${fullPath}' was updated. From ${outputValue(node.oldValue)} to ${outputValue(node.newValue)}`,
  ],
  nested: (node, fullPath, formattedChildren) => formattedChildren,
  default: () => [],
};

const formatNode = (node, path = '') => {
  const { key, type, children } = node;
  const fullPath = path === '' ? key : `${path}.${key}`;

  const formattedChildren = children
    ? children.flatMap(child => formatNode(child, fullPath))
    : [];

  const formatter = nodeFormatters[type] || nodeFormatters.default;
  return formatter(node, fullPath, formattedChildren);
};

const plainFormatDiff = (diff) => {
  const iter = (nodes) => {
    const lines = nodes.flatMap(node => formatNode(node)).join('\n');
    return lines;
  };

  return iter(diff);
};

export default plainFormatDiff;
