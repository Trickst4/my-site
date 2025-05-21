import _ from 'lodash';

const indentSize = 4;

const indent = depth => ' '.repeat((depth * indentSize)); // Определяем отступы с учетом смещения влево

const stringify = (value, depth) => {
  if (!_.isObject(value)) {
    return value;
  }
  const lines = Object.entries(value)
    .map(([key, val]) => `${indent(depth + 1)}${key}: ${stringify(val, depth + 1)}`);
  return `{\n${lines.join('\n')}\n${indent(depth)}}`;
};

const formatters = {
  nested: (node, depth, formattedChildren) => `${indent(depth)}${node.key}: {\n${formattedChildren}\n${indent(depth)}}`,
  unchanged: (node, depth) => `${indent(depth)}${node.key}: ${stringify(node.value, depth)}`,
  changed: (node, depth) => [
    `${indent(depth).slice(2)}- ${node.key}: ${stringify(node.oldValue, depth)}`,
    `${indent(depth).slice(2)}+ ${node.key}: ${stringify(node.newValue, depth)}`,
  ].join('\n'),
  removed: (node, depth) => `${indent(depth).slice(2)}- ${node.key}: ${stringify(node.value, depth)}`,
  added: (node, depth) => `${indent(depth).slice(2)}+ ${node.key}: ${stringify(node.value, depth)}`,
};

export const formatNode = (node, depth) => {
  const {
    key, type, children,
  } = node;

  if (type === undefined) {
    throw new Error(`Тип узла не определен для ключа: ${key}`);
  }

  const formattedChildren = children ? children.map(child => formatNode(child, depth + 1)).join('\n') : '';

  const formatter = formatters[type];
  if (!formatter) {
    throw new Error(`Неизвестный тип узла: ${type}`);
  }

  // Pass all necessary arguments to the formatter function
  return formatter(node, depth, formattedChildren);
};

const stylishFormatDiff = (diff) => {
  const iter = (nodes, depth) => {
    const lines = nodes.map(node => formatNode(node, depth));
    return `{\n${lines.join('\n')}\n}`;
  };

  return iter(diff, 1);
};

export default stylishFormatDiff;
