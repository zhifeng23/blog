---
layout: post
title: 压缩和解压缩字符串（Java实现）
tags: zipString 
categories: Java
---

<div class="toc"></div>

<br/>
----------
在进行实际项目开发时，有时候会限制传输的字符串的大小，这时候就会用到压缩字符串这种方案，下边记录一下java实现（考虑安全性）：

    /**
     * 压缩字符串
     * @param primStr  ：原字符串
     * @return ：压缩后字符串
     */
    public static String gzipString(String primStr)
    {
        if (primStr == null || primStr.length() < 5000) 
        { 
            log.warn("StringUtil.gzipString>>string is too small, no need to compress the string.");
            return primStr; 
        }
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        GZIPOutputStream gzip = null;
        try
        {
            gzip = new GZIPOutputStream(out);
            gzip.write(primStr.getBytes());
        }
        catch (IOException e)
        {
            log.error("StringUtil.gzipString>>compress String throw an exception!", e);
        }
        finally
        {
            if (gzip != null)
            {
                try
                {
                    gzip.close();
                }
                catch (IOException e)
                {
                    log.error("StringUtil.gzipString>>close Stream throw an exception!", e);
                }
            }
            if (out != null)
            {
                try
                {
                    out.close();
                }
                catch (IOException e)
                {
                    log.error("StringUtil.gzipString>>close Stream throw an exception!", e);
                }
            }
        }
        log.info("StringUtil.gzipString>>zip string successfully!");
        return new String(new Base64().encode(out.toByteArray()));
    }
    
    /**
     * 字符串解压缩
     * @param compressedStr ：已压缩的
     * @return ：解压缩后
     */
    public static String gunzipString(String compressedStr)
    {
        if (compressedStr == null) 
        { 
            return null;
        }
        
        Pattern pattern  = Pattern.compile("^([A-Za-z0-9+/]{4})*([A-Za-z0-9+/]{4}|[A-Za-z0-9+/]{3}=|[A-Za-z0-9+/]{2}==)$");
        Matcher matcher = pattern.matcher(compressedStr); 
        if (!matcher.matches())
        {
            return compressedStr;
        }
        
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        ByteArrayInputStream in = null;
        GZIPInputStream ginzip = null;
        byte[] compressed = null;
        String decompressed = null;
        try
        {
            compressed = new Base64().decode(compressedStr.getBytes());
            in = new ByteArrayInputStream(compressed);
            ginzip = new GZIPInputStream(in);
            byte[] buffer = new byte[1024];
            int offset = -1;
            while ((offset = ginzip.read(buffer)) != -1)
            {
                out.write(buffer, 0, offset);
            }
            decompressed = out.toString();
            log.info("StringUtil.gunzipString>>unzip string successfully!");
        }
        catch (IOException e)
        {
            log.error("StringUtil.gunzipString>>uncompression String throw an exception!", e);
        }
        finally
        {
            if (ginzip != null)
            {
                try
                {
                    ginzip.close();
                }
                catch (IOException e)
                {
                    log.error("StringUtil.gunzipString>>close Stream throw an exception!", e);
                }
            }
            if (in != null)
            {
                try
                {
                    in.close();
                }
                catch (IOException e)
                {
                    log.error("StringUtil.gunzipString>>close Stream throw an exception!", e);
                }
            }
            if (out != null)
            {
                try
                {
                    out.close();
                }
                catch (IOException e)
                {
                    log.error("StringUtil.gunzipString>>close Stream throw an exception!", e);
                }
            }
        }
        return decompressed;
    }
