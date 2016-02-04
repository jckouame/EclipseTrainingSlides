% title: Exploring CDT Core
% title_class:                      #empty, largeblend[123] or fullblend
% subtitle: The brain behind the C/C++ Editor
% subtitle_class:
% title_slide_class:
% title_slide_image:
% author: Marc-André Laperle
% thankyou_blend: largeblend3       #largeblend[123] or fullblend
% mail: marc-andre.laperle@ericsson.com
% lync: marc-andre.laperle@ericsson.com
% footer: Ericsson Internal
% footer: Rev PA1
% footer: 2015-12-15
% logoslide: false
% useBuilds: true
% animate: false         #animate logoslide (chrome only)
% aspect_ratio: 16:9     #16:9, 16:10 or 4:3

---
title: The C/C++ Editor

Has many features:

- Code completion
- Syntax and semantic coloring
- Source hover
- Refactoring
- Macro expansion
- Navigation
- Static code analysis
- Markers
- etc

<div style="position: absolute; left: 450px; top: 150px;">
<img src="images/cdt-editor.png" height="400" width="400"/>
</div>

---

title: Editor feature implementation

Most features use these frameworks:

- Editor framework from Platform UI
- AST: Abstract syntax tree (the 'physical' model)
- DOM: Document object model (the semantic model)
	- PDOM: Persistent DOM, aka the index

---

title: CODAN

**COD**e **AN**alysis

A static code analysis framework. A **checker** reports errors in the Problems view and editor.

Can be invoked in various ways:

- On demand (right-click Run analysis)
- On Open/Save in the editor (good for external tool integration)
- As you type (quite useful)
- Build (incremental, full)

---

title: Can be implemented at various approaches:

- File level ("plain text"): Use path of file to read line by line using Java APIs
- AST: Use CDT's AST to analyze the source file at a structural level.
- DOM: Use CDT's DOM to analyze at a semantic level.

Often AST and DOM(index) are used in combination, hence the class AbstractIndexAstChecker.
We will make use of all three approaches in the exercises.

---

title: Exercices

Project 1: We will create a checker that will warn if the file is longer than a set number of lines. This will only use normal Java libraries to analyze the code.

Project 2: We will create a checker that will warn if code is too complex. We will consider code too complex when it has deep nesting of if/else/for/while. This will use the AST.

Project 3: We will create a checker that will display an error if a method override another method but with the wrong return type.


---

title: Project 1

Description:

Create a checker that will warn if the file is longer than a set number of lines. This will only use normal Java libraries to analyze the code.

---

title: Project 1
subtitle: Step 1: Creating the checker skeleton

- Create a new plugin with cdt.codan.core dependency
- Create a new checker (extension), re-use AbstractCheckerWithProblemPreferences

---

title: Project 1
subtitle: Step 2:

- Check if it should report problem with shouldProduceProblems
- From the IFile, count the number of lines by reading lines one by one.
- Print total to console

Test it: Run it and check console

---

title: Project 1
subtitle: Step 3:

- Create a problem to will be reported (extension)
- Report the problem (when a certain number of lines is reached)

Test it: Run it and a warning should appear!

---

title: Project 1
subtitle: Step 4:

- Add a preference for the number of lines (addPreference). Use it
- Let the user know what the limit is. Use {0} in message to set parameter.

---

title: What is the AST?

Abstract Syntax Tree.

Tree of nodes representing the "physical" structure of a source file. Example:

<div style="float: left; background-color: #eeeeee; margin-left:50px; width:300px;">
#include <stdio.h><br/>
<br/>
int main() {<br/>
return 0;<br/>
}<br/>
<br/>
void foo() {<br/>
<br/>
}<br/>
</div>

<div style="float: right;">
<img src="images/dom.png" width="400"/>
</div>

---

title: Navigating the AST

2 ways to navigate the AST:

- get() methods
- Visitor pattern (the recommended way)

How to use a visitor:

- You create a visitor and specify what you would like to visit (types of nodes)
- When a desired node type is visited, the visit() method is called on your visitor with the node passed as argument
- Inside the visit method this is where the node can be analysed
- The visitor can choose to skip the children of that node or continue

---

title: AST Example

*Tiny example*

---

title: Project 2

Create a checker that will warn if code is too complex. We will consider code too complex when it has deep nesting of if/else/for/while. This time, we will make use of CDT's AST to analyze the code. We will be able to analyze based on nodes in a tree instead of plain text.

---

title: Project 2
subtitle: Step 1:

- Create a new checker (extension). This time we can use AbstractIndexAstChecker
- Open the AST view to explore the various node types. What should be useful?
- Create a visitor (extend ASTVisitor), make the root accept it. Visit only interesting nodes.

---

title: Project 2
subtitle: Step 2:

- Increment level of complexity when visiting a chosen node, decrement on leave
- Create and report the problem. Use the node's location to report at the correct line.

---

title: Project 2
subtitle: Step 3:

- Prevent more errors in deeper levels of children. I.e. do not "continue" after reporting
- Bonus: Create a preference for the complexity level

---

title: What is the DOM?

Document Object Model.

Each name in the AST has a "IBinding" that represents the semantics of the name.

AST + Bindings = DOM

Example of AST nodes vs binding: In a source file, printf is used in several places.

- Each individual call to printf is a node in the AST
- All printf nodes have a name that resolve to the same binding: an IFunction

*picture*

---

title: The index (PDOM)

DOM bindings are stored on disk to prevent re-parsing.

This data is called the Persistent DOM (PDOM) and often referred as just "the index".

- Parsing all files == indexing
- Quite fast reads, BTree
- Bindings have equivalent PDOM classes, example CFunction gets stored as PDOMCFunction
- Stores references to names and more. (Find references, Call hierarchy, etc)

---

title: Project 3

We will create a checker that will display an error if a method override another method but with the wrong return type.

Example: *picture or code*

---

title: Project 3
subtitle: Step 1:

- Create a new checker (extension). This time we can use AbstractIndexAstChecker
- Open the AST view to explore the various node types. What should be useful?
- Create a visitor (extend ASTVisitor), make the root accept it.

---
title: Project 3
subtitle: Approaches

There are multiple ways. 

Approach A

1. For each class name we find in the AST
2. Get its binding (ICPPClassType)
3. Get it's methods (ICPPMethod)
4. For each method, check for name return type conflict with the base methods.
5. Report error (where?)

We would like to report errors where the method declarators are but a ICPPMethod alone doesn't know about its location in the AST. We could find the location back from the index but we can save ourselves from this extra step.

---
title: Project 3

Approach B

1. For each method declarator node in the AST
2. Get its binding (ICPPMethod)
(That way, we have both the AST node of the method and it's semantic object)
3. Get its owner class (ICPPClassType)
4. Check for name return type conflict with the base method
5. Report the error at the location of the AST node

---
title: Project 3
subtitle: Step 2:

Find a potential method to analyze
- Visit only interesting nodes of the AST (declarators)
- Get its binding. Make sure it's a method and its owner

---
title: Project 3
subtitle: Step 3:

Find all potential conflicts
- Get the owner (class type) of the method
- Get **all** the base methods in the class. Cheat: Copy from ClassTypeHelper. Like ClassTypeHelper.getAllDeclaredMethods, but without getDeclaredMethods (the class own methods).

---
title: Project 3
subtitle: Step 4:

Detect the conflict
- Iterate through all base methods
- Check that the base method name is the same and is virtual
- Check that the return types are the same

---
title: Project 3
subtitle: Step 5:

- Create and report the problem. Use the node's location to report at the correct line.
- Tell the user which type it should be. ASTTypeUtil.getType should help

---
title: Code Examples
subtitle: Source Hover (CSourceHover.java)

~~~~{java}
public IStatus runOnAST(ILanguage lang, IASTTranslationUnit ast) {
	if (ast != null) {
		try {
			IASTNodeSelector nodeSelector = ast.getNodeSelector(null);
			if (fSelection.equals(Keywords.AUTO)) {
				...
			} else {
				IASTName name= nodeSelector.findEnclosingName(fTextRegion.getOffset(), fTextRegion.getLength());
				if (name != null) {
					IASTNode parent = name.getParent();
					if (parent instanceof ICPPASTTemplateId) {
						name = (IASTName) parent;
					}
					IBinding binding= name.resolveBinding();
					if (binding != null) {
						// Check for implicit names first, could be an implicit constructor call.
						if (name.getParent() instanceof IASTImplicitNameOwner) {
							IASTImplicitNameOwner implicitNameOwner = (IASTImplicitNameOwner) name.getParent();
							IASTName[] implicitNames = implicitNameOwner.getImplicitNames();
							if (implicitNames.length == 1) {
								IBinding implicitNameBinding = implicitNames[0].resolveBinding();
								if (implicitNameBinding instanceof ICPPConstructor) {
									binding = implicitNameBinding;
								}
							}
						}
~~~~


---

~~~~{java}

	if (binding instanceof IProblemBinding) {
		// Report problem as source comment.
		if (DEBUG) {
			IProblemBinding problem= (IProblemBinding) binding;
			fSource= "/* Problem:\n" + //$NON-NLS-1$
					" * " + problem.getMessage() +  //$NON-NLS-1$
					"\n */"; //$NON-NLS-1$
		}
	} else if (binding instanceof IMacroBinding) {
		fSource= computeSourceForMacro(ast, name, binding);
	} else if (binding instanceof IEnumerator) {
		// Show integer value for enumerators (bug 285126).
		fSource= computeSourceForEnumerator(ast, (IEnumerator) binding);
	} else {
		fSource= computeSourceForBinding(ast, binding);
	}
}

~~~~

---
title: Code Examples
subtitle: Implement method (ImplementMethodRefactoring.java)

~~~~{java}
private List<IASTSimpleDeclaration> findUnimplementedMethodDeclarations(IProgressMonitor pm) throws OperationCanceledException, CoreException {
	final SubMonitor sm = SubMonitor.convert(pm, 2);
	IASTTranslationUnit ast = getAST(tu, sm.newChild(1));
	final List<IASTSimpleDeclaration> list = new ArrayList<IASTSimpleDeclaration>();
	ast.accept(new ASTVisitor() {
		{ shouldVisitDeclarations = true; }
		@Override
		public int visit(IASTDeclaration declaration) {
			if (declaration instanceof IASTSimpleDeclaration) {
				IASTSimpleDeclaration simpleDeclaration = (IASTSimpleDeclaration) declaration;
				if (NodeHelper.isMethodDeclaration(simpleDeclaration)) {
					IASTDeclarator[] declarators = simpleDeclaration.getDeclarators();
					IBinding binding = declarators[0].getName().resolveBinding();
					if (isUnimplementedMethodBinding(binding, sm.newChild(0))) {
						list.add(simpleDeclaration);
						return ASTVisitor.PROCESS_SKIP;	
					}
				}
			}
			return ASTVisitor.PROCESS_CONTINUE;
		}
	});
	return list;
}

~~~~

---

~~~~{java}
private boolean isUnimplementedMethodBinding(IBinding binding, IProgressMonitor pm) {
	if (binding instanceof ICPPFunction) {
		if (binding instanceof ICPPMethod) {
			ICPPMethod methodBinding = (ICPPMethod) binding;
			if (methodBinding.isPureVirtual()) {
				return false; // Pure virtual not handled for now, see bug 303870
			}
		}
		
		try {
			return !DefinitionFinder.hasDefinition(binding, refactoringContext, pm);
		} catch (CoreException e) {
			CUIPlugin.log(e);
		}
	}
	
	return false;
}

~~~~

---

~~~~{java}
private IASTDeclaration createFunctionDefinition(IASTTranslationUnit unit, IASTSimpleDeclaration methodDeclaration, InsertLocation insertLocation) throws CoreException {
	IASTDeclSpecifier declSpecifier = methodDeclaration.getDeclSpecifier().copy(CopyStyle.withLocations);
	ICPPASTFunctionDeclarator functionDeclarator = (ICPPASTFunctionDeclarator) methodDeclaration.getDeclarators()[0];
	IASTNode declarationParent = methodDeclaration.getParent();
	String currentFileName = declarationParent.getNodeLocations()[0].asFileLocation().getFileName();
	if (Path.fromOSString(currentFileName).equals(insertLocation.getFile().getLocation())) {
		declSpecifier.setInline(true);
	}
...
	ICPPASTQualifiedName qName = createQualifiedNameFor(functionDeclarator, declarationParent, insertLocation);
	createdMethodDeclarator = nodeFactory.newFunctionDeclarator(qName);
	createdMethodDeclarator.setConst(functionDeclarator.isConst());
	createdMethodDeclarator.setRefQualifier(functionDeclarator.getRefQualifier());

~~~~

---
title: Code Examples
subtitle: Override markers (OverrideIndicatorManager.java)

~~~~{java}
class MethodFinder extends ASTVisitor {
	{
		shouldVisitDeclarators = true;
	}

	@Override
	public int visit(IASTDeclarator declarator) {
		if (!(declarator instanceof ICPPASTFunctionDeclarator)) {
			return PROCESS_CONTINUE;
		}
		IASTDeclarator decl = ASTQueries.findInnermostDeclarator(declarator);
		IASTName name = decl.getName();
		if (name != null) {
			IBinding binding = name.resolveBinding();
			if (binding instanceof ICPPMethod) {
				ICPPMethod method = (ICPPMethod) binding;
				try {
					ICPPMethod overriddenMethod = testForOverride(method, declarator);
					if (overriddenMethod != null) {
						try {
							ICElementHandle baseDeclaration = IndexUI.findAnyDeclaration(index, null, overriddenMethod);
							if (baseDeclaration == null) {
								ICElementHandle[] allDefinitions = IndexUI.findAllDefinitions(index, overriddenMethod);
								if (allDefinitions.length > 0) {
									baseDeclaration = allDefinitions[0];
								}
							}
							
							OverrideIndicator indicator = new OverrideIndicator(annotationKind, annotationMessage, baseDeclaration);
							
							IASTFileLocation fileLocation = declarator.getFileLocation();
							Position position = new Position(fileLocation.getNodeOffset(), fileLocation.getNodeLength());
							annotationMap.put(indicator, position);
						} catch (CoreException e) {
						}
					}
				} catch (DOMException e) {
				}
			}
		}
		return PROCESS_CONTINUE;
	}
~~~~

---

~~~~{java}
ICPPMethod overriddenMethod = testForOverride(method, declarator);
if (overriddenMethod != null) {
	try {
		ICElementHandle baseDeclaration = IndexUI.findAnyDeclaration(index, null, overriddenMethod);
		if (baseDeclaration == null) {
			ICElementHandle[] allDefinitions = IndexUI.findAllDefinitions(index, overriddenMethod);
			if (allDefinitions.length > 0) {
				baseDeclaration = allDefinitions[0];
			}
		}
		
		OverrideIndicator indicator = new OverrideIndicator(annotationKind, annotationMessage, baseDeclaration);
		
		IASTFileLocation fileLocation = declarator.getFileLocation();
		Position position = new Position(fileLocation.getNodeOffset(), fileLocation.getNodeLength());
		annotationMap.put(indicator, position);
~~~~

---
title: Code Examples
subtitle: Semantic highlighting (SemanticHighlightings.java)

~~~~{java}
/**
 * Semantic highlighting for namespaces.
 */
private static final class NamespaceHighlighting extends SemanticHighlightingWithOwnPreference {
	@Override
	public RGB getDefaultDefaultTextColor() {
		return RGB_BLACK;
	}
...
	@Override
	public boolean consumes(ISemanticToken token) {
		IBinding binding= token.getBinding();
		if (binding instanceof ICPPNamespace) {
			return true;
		}
		return false;
	}
}

~~~~
