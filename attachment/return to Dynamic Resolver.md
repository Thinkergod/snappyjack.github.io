#### return to Dynamic Resolver
����Ҫleak information��libc�汾


RELRO:Relocation Read Only
- No RELRO : ������ص�data structure��������д
- Partial RELRO : .dynamic ��.dynsym��.dynstr��ֻ�ܶ�(�����gccĬ��ֵ)
- Full RELRO�����е�symbol����ʱ�����Ѿ���ɣ�GOTֻ����û��link_map��resolver��ָ��

##### Leakless����Ҫ��
- ��Full ASLR��probgram�����memory layoutҪ��֪
- ͨ����information leakʱ��ʹ��DynELF��ȽϷ���

#### No RELRO
α��.dynstr
- readelf�ҳ�.dynamic��DT_STRTAB��λ��,���ı�����ֵ
- ��ԭ����.dynstrָ��һ���ɿ��Ƶ�buffer,��buffer�Ϸ�system���ִ�
- ��һ����û��resolver����symbol

α��`.rel.plt`��enrty 
- ��һ���ش��reloc_arg��ȥ,ʹ��.rel.plt+relog_arg���ڿɿصļ�������
- .dynstr + st_name������'system\0'