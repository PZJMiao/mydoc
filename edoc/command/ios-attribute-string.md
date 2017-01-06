iOS图文混排
===
iOS通过AttributedString实现图文混排

### 实现方法
``` ObjectiveC
+ (NSMutableAttributedString *)getSpannableString:(NSString *)string{
    
    NSInteger strLen=[string length];
    //【我是一张图片@图片名称 ,我是另一张图片@图片名称】 解析@图片名称 显示图文混排
    // Added By Plucky 2016-11-29 11:44
    NSString *regexStr=@"@\\w+";
    NSRegularExpression *regex = [NSRegularExpression regularExpressionWithPattern:regexStr options:NSRegularExpressionCaseInsensitive error:nil];
    NSArray * matches = [regex matchesInString:string options:0 range:NSMakeRange(0,strLen)];
    
    NSMutableArray *matchArr=[[NSMutableArray alloc]init];

    for (NSTextCheckingResult *match in matches) {
        NSInteger num=[match numberOfRanges];
        for (NSInteger i = 0; i < num; i++) {
            //以正则中的@,划分成不同的匹配部分
            NSRange aRange=[match rangeAtIndex:i];
            NSString *component = [string substringWithRange:aRange];
            [matchArr addObject:component];
        }
    }
    
    NSMutableAttributedString *attributedString=[[NSMutableAttributedString alloc]init];

    NSInteger count=matchArr.count;
    NSInteger lastIndex=0;
    for (NSInteger i=0; i<count; i++) {
        NSString *str=matchArr[i];
        NSRange range=[string rangeOfString:str];
        
        //取前面的字符串
        if (lastIndex>0) {
            NSString *preStr=[string substringWithRange:NSMakeRange(lastIndex, range.location-lastIndex)];
            [attributedString appendAttributedString:[[NSAttributedString alloc] initWithString:preStr]];
        }
        
        //记录上个图片的结尾位置
        lastIndex=range.location+range.length;
        
        NSInteger clen=str.length;
        NSString *imageStr=[str substringWithRange:NSMakeRange(1, clen-1)];
        NSTextAttachment *attch = [[NSTextAttachment alloc] init];
        attch.image = [UIImage imageNamed:imageStr];
        //attch.bounds = CGRectMake(0, 0, 20, 14);
        NSAttributedString *attchString = [NSAttributedString attributedStringWithAttachment:attch];
        [attributedString appendAttributedString:attchString];
        
        //获取结尾的字符串
        if (i==count-1&&lastIndex<strLen) {
            NSString *affStr=[string substringWithRange:NSMakeRange(lastIndex, strLen-lastIndex)];
            [attributedString appendAttributedString:[[NSAttributedString alloc] initWithString:affStr]];
        }
    }
    
    NSLog(@"Plucky -- SpannableString:%@",attributedString);
    return attributedString;
}
```
### 使用案例
``` ObjectiveC
NSMutableAttributedString *desc= [FHLib getSpannableString:[_waybill getRichUserInfo]];
[_labelUser setAttributedText:desc];
```