<!--yml

category: 未分类

date: 2024-05-27 15:02:24

-->

# 介绍 | Codemodder

> 来源：[https://codemodder.io/](https://codemodder.io/)

```
 @Codemod(  id =  "pixee:java/secure-random",  reviewGuidance =  ReviewGuidance.MERGE_WITHOUT_REVIEW)  public  final  class  SecureRandomCodemod  extends  SarifPluginJavaParserChanger<ObjectCreationExpr>  {  private  static  final  String  DETECTION_RULE  =  """ rules: - id: secure-random 
 pattern: new Random()
 """;  @Inject  public  SecureRandomCodemod(@SemgrepScan(yaml =  DETECTION_RULE)  RuleSarif sarif)  {  super(sarif,  ObjectCreationExpr.class);  }  @Override  public  boolean  onResultFound(  final  CodemodInvocationContext context,  final  CompilationUnit cu,  final  ObjectCreationExpr objectCreationExpr,  final  Result result)  {  objectCreationExpr.setType("SecureRandom");  addImportIfMissing(cu,  SecureRandom.class.getName());  return  true;  }  } 
```
