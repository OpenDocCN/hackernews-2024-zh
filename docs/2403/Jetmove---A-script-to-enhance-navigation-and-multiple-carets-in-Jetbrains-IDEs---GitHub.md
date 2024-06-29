<!--yml

category: 未分类

date: 2024-05-29 12:31:52

-->

# Jetmove - 一个用于增强Jetbrains IDE中导航和多个插入符的脚本 · GitHub

> 来源：[https://gist.github.com/lurebat/df773fecbc6829625d996fc8a65d5e25](https://gist.github.com/lurebat/df773fecbc6829625d996fc8a65d5e25)

|  | import com.intellij.find.FindModel |
| --- | --- |
|  | import com.intellij.openapi.actionSystem.AnActionEvent |
|  | import com.intellij.openapi.actionSystem.DataContext |
|  | import com.intellij.openapi.editor.Caret |
|  | import com.intellij.openapi.editor.Editor |
|  | import com.intellij.openapi.editor.actionSystem.TypedAction |
|  | import com.intellij.openapi.editor.actionSystem.TypedActionHandler |
|  | import com.intellij.openapi.ui.DialogPanel |
|  | import com.intellij.openapi.ui.DialogWrapper |
|  | import com.intellij.openapi.util.TextRange |
|  | import com.intellij.ui.EditorTextField |
|  | import com.intellij.ui.dsl.builder.bind |
|  | import com.intellij.ui.dsl.builder.bindSelected |
|  | import com.intellij.ui.dsl.builder.panel |
|  | import com.intellij.ui.dsl.builder.toMutableProperty |
|  | import liveplugin.currentEditor |
|  | import liveplugin.editor |
|  | import liveplugin.executeCommand |
|  | import liveplugin.registerAction |
|  | import org.intellij.lang.regexp.RegExpFileType |
|  | import javax.swing.JComponent |
|  |  |
|  | /** |
|  | * 表示文档中提供范围的通用文本对象。 |
|  | */ |
|  | interface TextObject { |
|  | val name: String |
|  | val shortcut: String |
|  |  |
|  | /** |
|  | * 计算当前文本对象的范围。 |
|  | * @param inside 是否在文本对象内部或周围查找范围。 |
|  | * @param caret 作为参考点使用的插入符。 |
|  | * @return 表示范围起始和结束索引的一对，如果不适用则返回null。 |
|  | */ |
|  | fun range(inside: Boolean, caret: Caret): Pair<Int, Int>? |
|  | } |
|  |  |
|  | /** |
|  | * 简化版TextObject，更容易创建特定文本对象。 |
|  | */ |
|  | abstract class SimpleTextObject : TextObject { |
|  | override fun range(inside: Boolean, caret: Caret): Pair<Int, Int>? = |
|  | range(inside, caret.editor.document.charsSequence, caret.offset) |
|  |  |
|  | /** |
|  | * 基于文档中的字符和偏移量计算文本对象的范围。 |
|  | * @param inside 是否在文本对象内部或周围查找范围。 |
|  | * @param chars 要分析的字符序列。 |
|  | * @param offset 要考虑的字符序列内的偏移量。 |
|  | * @return 表示范围起始和结束索引的一对，如果不适用则返回null。 |
|  | */ |
|  | abstract fun range(inside: Boolean, chars: CharSequence, offset: Int): Pair<Int, Int>? |
|  | } |
|  |  |
|  | class CharPairs(private val starts: CharArray, private val ends: CharArray) { |
|  | constructor(vararg pairs: Pair<Char, Char>) : this( |
|  | pairs.map { it.first }.toCharArray(), |
|  | pairs.map { it.second }.toCharArray() |
|  | ) |
|  |  |
|  | fun findPreviousStart(text: CharSequence, offset: Int): Int? { |
|  | val charsCount = Array(starts.size) { 0 } |
|  | for (i in offset - 1 downTo 0) { |
|  | val char = text[i] |
|  | val start = starts.indexOf(char) |
|  | if (start != -1) { |
|  | if (charsCount[start] == 0) return i |
|  | charsCount[start]-- |
|  | continue |
|  | } |
|  | val end = ends.indexOf(char) |
|  | if (end != -1) { |
|  | charsCount[end]++ |
|  | } |
|  | } |
|  | return null |
|  | } |
|  |  |
|  | fun findNextEnd(text: CharSequence, offset: Int): Int? { |
|  | val charsCount = Array(ends.size) { 0 } |
|  | for (i in offset until text.length) { |
|  | val char = text[i] |
|  | val end = ends.indexOf(char) |
|  | if (end != -1) { |
|  | if (charsCount[end] == 0) return i |
|  | charsCount[end]-- |
|  | continue |
|  | } |
|  | val start = starts.indexOf(char) |
|  | if (start != -1) { |
|  | charsCount[start]++ |
|  | } |
|  | } |
|  | return null |
|  | } |
|  |  |
|  | fun findNextStart(text: CharSequence, offset: Int): Int? = text.indexOfAny(starts, offset).takeIf { it != -1 } |
|  | } |
|  |  |
|  | class BalancedTextObject(override val name: String, override val shortcut: String, private val pairs: CharPairs) : |
|  | SimpleTextObject() { |
|  | constructor(name: String, shortcut: String, vararg pairs: Pair<Char, Char>) : this( |
|  | name, |
|  | shortcut, |
|  | CharPairs(*pairs) |
|  | ) |
|  |  |
|  | override fun range(inside: Boolean, caret: CharSequence, offset: Int): Pair<Int, Int>? { |
|  | val start = pairs.findPreviousStart(caret, offset) ?: pairs.findNextStart(caret, offset) ?: return null |
|  | val end = pairs.findNextEnd(caret, offset) ?: return null |
|  | return if (inside) Pair(start + 1, end) else Pair(start, end + 1) |
|  | } |
|  | } |
|  |  |
|  | fun getUnmatchedCharIndex(isBackwards: Boolean, text: CharSequence, char: Char, match: Char): Int { |
|  | var count = 0 |
|  | val range = if (isBackwards) text.indices.reversed() else text.indices |
|  | for (i in range) { |
|  | val c = text[i] |
|  | if (c == char) count++ |
|  | if (c == match) { |
|  | if (count == 0) return i |
|  | count-- |
|  | } |
|  | } |
|  | return -1 |
|  | } |
|  |  |
|  | val textObjects = arrayOf( |
|  | BalancedTextObject("word", "W", ' ' to ' ', ' ' to '\n', '\n' to ' '), |
|  | BalancedTextObject("double-quote", "shift QUOTE'", '"' to '"'), |
|  | BalancedTextObject("single-quote", "QUOTE", '\'' to '\''), |
|  | BalancedTextObject("paren", "shift 9", '(' to ')'), |
|  | BalancedTextObject("bracket", "[", '[' to ']'), |
|  | BalancedTextObject("brace", "shift [", '{' to '}'), |
|  | BalancedTextObject("angle-bracket", "shift ,", '<' to '>'), |
|  | BalancedTextObject("backtick", "`", '`' to '`'), |
|  | BalancedTextObject("block", "B", '{' to '}', '(' to ')', '[' to ']'), |
|  | ) |
|  |  |
|  |  |
|  | val baseShortcutAround = "alt S, " |
|  | val baseShortcutInside = "alt shift S," |
|  |  |
|  | fun registerTextObjectActions() { |
|  | for (textObject in textObjects) { |
|  | registerAction("text-object-around-${textObject.name}", baseShortcutAround + textObject.shortcut) { |
|  | val editor = it.project?.currentEditor ?: return@registerAction |
|  |  |
|  | editor.caretModel.allCarets.forEach { caret -> |
|  | val range = textObject.range(false, caret) |
|  | if (range != null) { |
|  | caret.setSelection(range.first, range.second) |
|  | } |
|  | } |
|  | } |
|  | registerAction("text-object-inside-${textObject.name}", baseShortcutInside + textObject.shortcut) { |
|  | val editor = it.project?.currentEditor ?: return@registerAction |
|  | editor.caretModel.allCarets.forEach { caret -> |
|  | val range = textObject.range(true, caret) |
|  | if (range != null) { |
|  | caret.setSelection(range.first, range.second) |
|  | } |
|  | } |
|  | } |
|  | } |
|  | } |
|  |  |
|  | data class Model( |
|  | var onChanged: () -> Unit = {}, |
|  | ) : FindModel() { |
|  | fun matches(text: String): Boolean { |
|  | val re = compileRegExp() |
|  | return re.matcher(text).find() |
|  | } |
|  |  |
|  | var string |
|  | get() = stringToFind |
|  | set(value) { |
|  | stringToFind = value |
|  | onChanged() |
|  | } |
|  | var keep = true |
|  | set(value) { |
|  | field = value |
|  | onChanged() |
|  | } |
|  | var case = super.isCaseSensitive() |
|  | set(value) { |
|  | field = value |
|  | super.setCaseSensitive(value) |
|  | onChanged() |
|  | } |
|  |  |
|  | var words = super.isWholeWordsOnly() |
|  | set(value) { |
|  | field = value |
|  | super.setWholeWordsOnly(value) |
|  | onChanged() |
|  | } |
|  |  |
|  | var regex = super.isRegularExpressions() |
|  | set(value) { |
|  |  |
|  | field = value |
|  | super.setRegularExpressions(value) |
|  | onChanged() |
|  | } |
|  |  |
|  | } |
|  |  |
|  | fun matchingCarets(model: Model, carets: List<Caret>): List<Pair<Caret, ClosedRange<Int>?>> { |
|  | val re = if (model.regex) { |
|  | model.compileRegExp().toRegex() |
|  | } else { |
|  | Regex(model.stringToFind.let { |
|  | if (model.words) it.split("\\s+").joinToString("&#124;") { "\\b$it\\b" } else it |
|  | }, if (model.case) RegexOption.IGNORE_CASE else RegexOption.LITERAL) |
|  | } |
|  | return carets.map { |
|  | // get selected text or line |
|  | val selectedText = it.selectedText?.ifEmpty { null } ?: getLine(it) |
|  | it to if (model.keep) { |
|  | re.find(selectedText)?.range |
|  | } else { |
|  | (if (re.find(selectedText) == null) 0..selectedText.length else null) |
|  | } |
|  | } |
|  |  |
|  | } |
|  |  |
|  | fun filterCarets(it: AnActionEvent, keep: Boolean) { |
|  | val editor = it.editor ?: return |
|  | val project = it.project ?: return |
|  | val model = Model() |
|  | var panel: DialogPanel? = null |
|  | model.regex = true |
|  | model.keep = keep |
|  |  |
|  | val wrapper = object : DialogWrapper(editor.contentComponent, true) { |
|  | init { |
|  | init() |
|  | title = "Filter Caret" |
|  | } |
|  |  |
|  |  |
|  | override fun 创建中心面板(): JComponent { |
|  | 面板 = 面板 { |
|  | 行 { |
|  | 标签("过滤：") |
|  | 单元格(EditorTextField(project, RegExpFileType.INSTANCE).apply { |
|  | 设置单行模式(true) |
|  | setPreferredWidth(300) |
|  | }).bind( |
|  | { it.text }, |
|  | { e, s -> e.text = s }, |
|  | model::string.toMutableProperty() |
|  | ).focused() |
|  | } |
|  | 按钮组("动作") { |
|  | 行 { |
|  | 单选按钮("保留", true) |
|  | 单选按钮("删除", false) |
|  | } |
|  | }.bind(model::keep.toMutableProperty()) |
|  | 行 { |
|  | 复选框("区分大小写").bindSelected( |
|  | model::case.toMutableProperty() |
|  | ) |
|  | 复选框("单词").bindSelected(model::words.toMutableProperty()) |
|  | 复选框("正则表达式").bindSelected(model::regex.toMutableProperty()) |
|  | } |
|  | } |
|  |  |
|  | 返回面板!! |
|  | } |
|  | } |
|  |  |
|  | 如果 (!wrapper.showAndGet()) return |
|  |  |
|  |  |
|  | val 插入符模型 = editor.caretModel |
|  |  |
|  | 匹配插入符(model, editor.caretModel.allCarets).filter { it.second == null }.forEach { (c, _) -> |
|  | 插入符模型.removeCaret(c) |
|  | } |
|  | } |
|  |  |
|  |  |
|  | registerAction("过滤插入符-保留", "alt K") { filterCarets(it, true) } |
|  | registerAction("过滤插入符-删除", "alt shift K") { filterCarets(it, false) } |
|  |  |
|  |  |
|  |  |
|  | 内部对象 EditorKeyListener : TypedActionHandler { |
|  | 私有的 val action = TypedAction.getInstance() |
|  | 私有的 val 附加 = mutableMapOf<Editor, TypedActionHandler>() |
|  | 私有的 var 原始处理程序: TypedActionHandler? = null |
|  |  |
|  | override fun 执行(editor: Editor, charTyped: Char, dataContext: DataContext) { |
|  | (attached[editor] ?: originalHandler ?: return).execute(editor, charTyped, dataContext) |
|  | } |
|  |  |
|  | fun 附加(editor: Editor, callback: TypedActionHandler) { |
|  | 如果 (附加.isEmpty()) { |
|  | 原始处理程序 = action.rawHandler |
|  | action.setupRawHandler(this) |
|  | } |
|  |  |
|  | 附加[editor] = callback |
|  | } |
|  |  |
|  | fun 分离(editor: Editor) { |
|  | 从附加中删除(editor) |
|  |  |
|  | 如果 (附加.isEmpty()) { |
|  | 原始处理程序?.let(action::setupRawHandler) |
|  | 原始处理程序 = null |
|  | } |
|  | } |
|  | } |
|  |  |
|  | fun Editor.等待字符(callback: (Char) -> Unit) { |
|  | EditorKeyListener.附加( |
|  | this |
|  | ) { editor, char, dataContext -> |
|  | EditorKeyListener.分离(editor) |
|  | callback(char) |
|  | } |
|  | } |
|  |  |
|  | fun 选择至字符(editor: Editor, backwards: Boolean) { |
|  | val 文档 = editor.document |
|  | val 文本 = document.charsSequence |
|  |  |
|  | editor.等待字符 { char -> |
|  | val 所有插入符 = editor.caretModel.allCarets |
|  | 如果 (所有插入符.size != 1) { |
|  | 所有插入符号.forEach { 插入符 -> |
|  | // 搜索直到换行符 |
|  | val 新行 = text.indexOf('\n', caret.offset) |
|  | 如果 (向后) { // 向后搜索 |
|  | val index = text.lastIndexOf(char, if (newLine == -1) text.length else newLine - 1) |
|  | if (index != -1) { |
|  | caret.setSelection(index, caret.selectionEnd) |
|  | } |
|  | } else { // 向前搜索 |
|  | val index = text.indexOf(char, caret.offset) |
|  | if (index != -1 && index < newLine) { |
|  | caret.setSelection(caret.selectionStart, index + 1) |
|  | } |
|  | } |
|  | } |
|  | } else { |
|  | val caret = allCarets[0] |
|  | val offset = if (backwards) caret.selectionStart else caret.selectionEnd |
|  | val index = if (backwards) text.lastIndexOf(char, offset - 1) else text.indexOf(char, offset + 1) |
|  | if (index != -1) { |
|  | caret.setSelection( |
|  | if (backwards) index else caret.selectionStart, |
|  | if (backwards) caret.selectionEnd else index + 1 |
|  | ) |
|  | } |
|  | } |
|  | } |
|  | } |
|  |  |
|  | registerAction("向前选择", "alt X") { |
|  | val editor = it.editor ?: return@registerAction |
|  | selectToChar(editor, false) |
|  | } |
|  |  |
|  | registerAction("向后选择", "alt shift X") { |
|  | val editor = it.editor ?: return@registerAction |
|  | selectToChar(editor, true) |
|  | } |
|  |  |
|  | registerAction("切换选择", "alt Z") { |
|  | val editor = it.editor ?: return@registerAction |
|  | val caretModel = editor.caretModel |
|  | val allCarets = caretModel.allCarets |
|  | allCarets.forEach { caret -> |
|  | if (caret.offset < caret.leadSelectionOffset) { |
|  | caret.moveToOffset(caret.selectionEnd) |
|  | } else { |
|  | caret.moveToOffset(caret.selectionStart) |
|  | } |
|  | } |
|  | } |
|  |  |
|  | 变量 bracketPairs = mapOf( |
|  | '(' 转为 ')', |
|  | '[' 转为 ']', |
|  | '{' 转为 '}', |
|  | '<' 转为 '>', |
|  | '"' 转为 '"', |
|  | '\'' 转为 '\'', |
|  | '`' 转为 '`', |
|  | ')' 转为 '(', |
|  | ']' 转为 '[', |
|  | '}' 转为 '{', |
|  | '>' 转为 '<', |
|  | ) |
|  |  |
|  |  |
|  |  |
|  | registerAction("替换括号", "alt shift 9") { ev -> |
|  | val editor = ev.editor ?: return@registerAction |
|  | val project = ev.project ?: return@registerAction |
|  | val caretModel = editor.caretModel |
|  | val allCarets = caretModel.allCarets |
|  |  |
|  | editor.waitForChar { userChar -> |
|  | val userOpposite = if (userChar == '-') ('-') else (bracketPairs[userChar] ?: return@waitForChar) |
|  | allCarets.forEach { caret -> |
|  | // 查找下一个或当前括号 |
|  | val text = editor.document.charsSequence.substring(caret.offset) |
|  |  |
|  | text.indexOfAny(bracketPairs.keys.toCharArray()).takeIf { it != -1 }?.let { index -> |
|  | val char = text[index] |
|  | val charOpposite = bracketPairs[char] ?: return@forEach |
|  | val charOppositeIndex = getUnmatchedCharIndex(false, text.substring(index + 1), char, charOpposite) + index + 1 |
|  |  |
|  | if (charOppositeIndex != -1) { |
|  | if (userChar == '-') { |
|  | // 删除括号 |
|  | editor.document.executeCommand( |
|  | 项目, |
|  | 删除括号 - $char $charOpposite |
|  | ) { |
|  | editor.document.deleteString(caret.offset + index, caret.offset + index + 1) |
|  | editor.document.deleteString( |
|  | caret.offset + charOppositeIndex - 1, |
|  | caret.offset + charOppositeIndex |
|  | ) |
|  | } |
|  | } else { |
|  | editor.document.executeCommand( |
|  | project, |
|  | "Replace brackets - $char $charOpposite - with $userChar $userOpposite" |
|  | ) { |
|  | // replace char with userChar and charOpposite with userOpposite |
|  | editor.document.replaceString( |
|  | caret.offset + index, |
|  | caret.offset + index + 1, |
|  | userChar.toString() |
|  | ) |
|  | editor.document.replaceString( |
|  | caret.offset + charOppositeIndex, |
|  | caret.offset + charOppositeIndex + 1, |
|  | userOpposite.toString() |
|  | ) |
|  | } |
|  | } |
|  | } |
|  | } |
|  | } |
|  | } |
|  |  |
|  |  |
|  | } |
|  |  |
|  | registerTextObjectActions() |
|  |  |
|  | fun getLine(it: Caret): String { |
|  | val line = it.editor.document.getLineNumber(it.offset) |
|  | val start = it.editor.document.getLineStartOffset(line) |
|  | val end = it.editor.document.getLineEndOffset(line) |
|  | return it.editor.document.getText(TextRange(start, end)) |
|  | } |
