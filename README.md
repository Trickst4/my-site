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

const formatNode = (node, path = '') => {
  const {
    key, type, value, oldValue, newValue, children,
  } = node;

  const fullPath = path === '' ? key : `${path}.${key}`;
  // Так и не понял какую функцию надо изменить, поэтому строчку выше оставил как есть

  const formattedChildren = children
    ? children.flatMap(child => formatNode(child, fullPath))
    : [];

  // Диспетчеризация по ключу вместо switch
  const formatters = {
    added: () => [`Property '${fullPath}' was added with value: ${outputValue(value)}`],
    removed: () => [`Property '${fullPath}' was removed`],
    changed: () => [`Property '${fullPath}' was updated. From ${outputValue(oldValue)} to ${outputValue(newValue)}`],
    nested: () => formattedChildren,
    // Если тип не найден, возвращаем пустой массив
    default: () => [],
  };

  return (formatters[type] || formatters.default)();
};

const plainFormatDiff = (diff) => {
  const iter = (nodes) => {
    const lines = nodes.flatMap(node => formatNode(node)).join('\n');
    return lines;
  };

  return iter(diff);
};

export default plainFormatDiff;
