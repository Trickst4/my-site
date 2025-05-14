import * as fs from 'fs'
import path, { dirname } from 'path'
import { fileURLToPath } from 'url'
import getDiff from '../src/index.js'
import getParseFile from '../src/parse.js'
import selectFormatter from '../src/formatters/index.js'

const __filename = fileURLToPath(import.meta.url)
const __dirname = dirname(__filename)
const getFixturePath = filename => path.join(__dirname, '..', '__fixtures__', filename)
const readFile = filename => fs.readFileSync(getFixturePath(filename), 'utf-8').trim()

describe('Error handling tests', () => {
  test('Throws error on unsupported file type', () => {
    expect(() => getParseFile(getFixturePath('expectedResult.tx'))).toThrow()
  })

  test('Throws error on unsupported formatter type in getDiff', () => {
    expect(() => getDiff(getFixturePath('file1.json'), getFixturePath('file2.json'), 'uncorrectType')).toThrow()
  })

  test('Throws error on unsupported formatter selection', () => {
    const unsupported = 'uncorrectType'
    expect(() => selectFormatter(unsupported)).toThrow(`Formatter "${unsupported}" is not supported`)
  })
})

describe('Diff output tests', () => {
  const filePairs = [
    ['file1.json', 'file2.json', 'expectedResult.txt', undefined],
    ['file1.yml', 'file2.yml', 'expectedResult.txt', undefined],
  ]

  test.each(filePairs)(
    'Compare %s and %s files with default formatter',
    (file1, file2, expectedFile, format) => {
      const result = getDiff(getFixturePath(file1), getFixturePath(file2), format)
      const expected = readFile(expectedFile)
      expect(result.trim()).toEqual(expected)
    }
  )
})

describe('Diff output with specific formatters', () => {
  const formatTests = [
    ['stylish', 'expectedResult.txt'],
    ['plain', 'expectedResultPlain.txt'],
    ['json', 'expectedResultJson.txt'],
  ]

  test.each(formatTests)(
    'Compare JSON files with %s formatter',
    (format, expectedFile) => {
      const result = getDiff(getFixturePath('file1.json'), getFixturePath('file2.json'), format)
      const expected = readFile(expectedFile)
      expect(result.trim()).toEqual(expected)
    }
  )
})
