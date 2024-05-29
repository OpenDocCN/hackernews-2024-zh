<!--yml
category: 未分类
date: 2024-05-29 12:31:52
-->

# Jetmove - A script to enhance navigation and multiple carets in Jetbrains IDEs · GitHub

> 来源：[https://gist.github.com/lurebat/df773fecbc6829625d996fc8a65d5e25](https://gist.github.com/lurebat/df773fecbc6829625d996fc8a65d5e25)

|  | import com.intellij.find.FindModel |
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
|  | * Represents a generic text object that provides a range within a document. |
|  | */ |
|  | interface TextObject { |
|  | val name: String |
|  | val shortcut: String |
|  |  |
|  | /** |
|  | * Calculates the range for the current text object. |
|  | * @param inside Whether to find the range inside or around the text object. |
|  | * @param caret The caret to use as a reference point. |
|  | * @return A pair representing the start and end index of the range, or null if not applicable. |
|  | */ |
|  | fun range(inside: Boolean, caret: Caret): Pair<Int, Int>? |
|  | } |
|  |  |
|  | /** |
|  | * A simplified version of TextObject for easier creation of specific text objects. |
|  | */ |
|  | abstract class SimpleTextObject : TextObject { |
|  | override fun range(inside: Boolean, caret: Caret): Pair<Int, Int>? = |
|  | range(inside, caret.editor.document.charsSequence, caret.offset) |
|  |  |
|  | /** |
|  | * Calculates the text object's range based on characters and an offset within a document. |
|  | * @param inside Whether to find the range inside or around the text object. |
|  | * @param chars The character sequence to analyze. |
|  | * @param offset The offset within the character sequence to consider. |
|  | * @return A pair representing the start and end index of the range, or null if not applicable. |
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
|  | override fun createCenterPanel(): JComponent { |
|  | panel = panel { |
|  | row { |
|  | label("Filter:") |
|  | cell(EditorTextField(project, RegExpFileType.INSTANCE).apply { |
|  | setOneLineMode(true) |
|  | setPreferredWidth(300) |
|  | }).bind( |
|  | { it.text }, |
|  | { e, s -> e.text = s }, |
|  | model::string.toMutableProperty() |
|  | ).focused() |
|  | } |
|  | buttonsGroup("Action") { |
|  | row { |
|  | radioButton("Keep", true) |
|  | radioButton("Remove", false) |
|  | } |
|  | }.bind(model::keep.toMutableProperty()) |
|  | row { |
|  | checkBox("Match case").bindSelected( |
|  | model::case.toMutableProperty() |
|  | ) |
|  | checkBox("Words").bindSelected(model::words.toMutableProperty()) |
|  | checkBox("Regex").bindSelected(model::regex.toMutableProperty()) |
|  | } |
|  | } |
|  |  |
|  | return panel!! |
|  | } |
|  | } |
|  |  |
|  | if (!wrapper.showAndGet()) return |
|  |  |
|  |  |
|  | val caretModel = editor.caretModel |
|  |  |
|  | matchingCarets(model, editor.caretModel.allCarets).filter { it.second == null }.forEach { (c, _) -> |
|  | caretModel.removeCaret(c) |
|  | } |
|  | } |
|  |  |
|  |  |
|  | registerAction("filter-carets-keep", "alt K") { filterCarets(it, true) } |
|  | registerAction("filter-carets-remove", "alt shift K") { filterCarets(it, false) } |
|  |  |
|  |  |
|  |  |
|  | internal object EditorKeyListener : TypedActionHandler { |
|  | private val action = TypedAction.getInstance() |
|  | private val attached = mutableMapOf<Editor, TypedActionHandler>() |
|  | private var originalHandler: TypedActionHandler? = null |
|  |  |
|  | override fun execute(editor: Editor, charTyped: Char, dataContext: DataContext) { |
|  | (attached[editor] ?: originalHandler ?: return).execute(editor, charTyped, dataContext) |
|  | } |
|  |  |
|  | fun attach(editor: Editor, callback: TypedActionHandler) { |
|  | if (attached.isEmpty()) { |
|  | originalHandler = action.rawHandler |
|  | action.setupRawHandler(this) |
|  | } |
|  |  |
|  | attached[editor] = callback |
|  | } |
|  |  |
|  | fun detach(editor: Editor) { |
|  | attached.remove(editor) |
|  |  |
|  | if (attached.isEmpty()) { |
|  | originalHandler?.let(action::setupRawHandler) |
|  | originalHandler = null |
|  | } |
|  | } |
|  | } |
|  |  |
|  | fun Editor.waitForChar(callback: (Char) -> Unit) { |
|  | EditorKeyListener.attach( |
|  | this |
|  | ) { editor, char, dataContext -> |
|  | EditorKeyListener.detach(editor) |
|  | callback(char) |
|  | } |
|  | } |
|  |  |
|  | fun selectToChar(editor: Editor, backwards: Boolean) { |
|  | val document = editor.document |
|  | val text = document.charsSequence |
|  |  |
|  | editor.waitForChar { char -> |
|  | val allCarets = editor.caretModel.allCarets |
|  | if (allCarets.size != 1) { |
|  | allCarets.forEach { caret -> |
|  | // search until newline |
|  | val newLine = text.indexOf('\n', caret.offset) |
|  | if (backwards) { // search backwards |
|  | val index = text.lastIndexOf(char, if (newLine == -1) text.length else newLine - 1) |
|  | if (index != -1) { |
|  | caret.setSelection(index, caret.selectionEnd) |
|  | } |
|  | } else { // search forwards |
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
|  | registerAction("select-to-forwards", "alt X") { |
|  | val editor = it.editor ?: return@registerAction |
|  | selectToChar(editor, false) |
|  | } |
|  |  |
|  | registerAction("select-to-backwards", "alt shift X") { |
|  | val editor = it.editor ?: return@registerAction |
|  | selectToChar(editor, true) |
|  | } |
|  |  |
|  | registerAction("switch-selection", "alt Z") { |
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
|  | var bracketPairs = mapOf( |
|  | '(' to ')', |
|  | '[' to ']', |
|  | '{' to '}', |
|  | '<' to '>', |
|  | '"' to '"', |
|  | '\'' to '\'', |
|  | '`' to '`', |
|  | ')' to '(', |
|  | ']' to '[', |
|  | '}' to '{', |
|  | '>' to '<', |
|  | ) |
|  |  |
|  |  |
|  |  |
|  | registerAction("replace-brackets", "alt shift 9") { ev -> |
|  | val editor = ev.editor ?: return@registerAction |
|  | val project = ev.project ?: return@registerAction |
|  | val caretModel = editor.caretModel |
|  | val allCarets = caretModel.allCarets |
|  |  |
|  | editor.waitForChar { userChar -> |
|  | val userOpposite = if (userChar == '-') ('-') else (bracketPairs[userChar] ?: return@waitForChar) |
|  | allCarets.forEach { caret -> |
|  | // find next or current bracket |
|  | val text = editor.document.charsSequence.substring(caret.offset) |
|  |  |
|  | text.indexOfAny(bracketPairs.keys.toCharArray()).takeIf { it != -1 }?.let { index -> |
|  | val char = text[index] |
|  | val charOpposite = bracketPairs[char] ?: return@forEach |
|  | val charOppositeIndex = getUnmatchedCharIndex(false, text.substring(index + 1), char, charOpposite) + index + 1 |
|  |  |
|  | if (charOppositeIndex != -1) { |
|  | if (userChar == '-') { |
|  | // Delete brackets |
|  | editor.document.executeCommand( |
|  | project, |
|  | "Delete brackets - $char $charOpposite" |
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