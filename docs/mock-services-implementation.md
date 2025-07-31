# Mock Services Implementation - SRMP

## Overview

This document provides detailed mock service implementations for all external dependencies, enabling offline development and testing.

## 1. WeChat Work Mock Service

### Full Implementation

```typescript
// packages/utils/src/lib/mocks/wechat-work-mock.service.ts
import { Injectable } from '@nestjs/common';
import { v4 as uuidv4 } from 'uuid';

export interface WeChatUser {
  userId: string;
  name: string;
  avatar: string;
  department: string[];
  mobile?: string;
  email?: string;
}

export interface WeChatNotification {
  touser: string;
  msgtype: 'text' | 'markdown' | 'news';
  agentid: number;
  text?: { content: string };
  markdown?: { content: string };
}

export interface WeChatOAuthResult {
  access_token: string;
  expires_in: number;
  refresh_token: string;
  openid: string;
  scope: string;
  unionid: string;
}

@Injectable()
export class WeChatWorkMockService {
  private mockUsers = new Map<string, WeChatUser>();
  private mockTokens = new Map<string, string>();
  private notificationQueue: any[] = [];
  private accessToken = 'mock-access-token-' + Date.now();

  constructor() {
    this.initializeMockData();
  }

  private initializeMockData() {
    // Create mock users
    const mockUserData = [
      {
        userId: 'student001',
        name: '张三',
        avatar: 'https://api.dicebear.com/7.x/avataaars/svg?seed=student001',
        department: ['计算机科学系'],
        mobile: '13800138001',
        email: 'zhangsan@university.edu.cn'
      },
      {
        userId: 'professor001',
        name: '李教授',
        avatar: 'https://api.dicebear.com/7.x/avataaars/svg?seed=professor001',
        department: ['计算机科学系', '人工智能研究所'],
        email: 'li.professor@university.edu.cn'
      },
      {
        userId: 'secretary001',
        name: '王秘书',
        avatar: 'https://api.dicebear.com/7.x/avataaars/svg?seed=secretary001',
        department: ['教务处'],
        email: 'wang.secretary@university.edu.cn'
      }
    ];

    mockUserData.forEach(user => {
      this.mockUsers.set(user.userId, user);
      // Create mock tokens for each user
      this.mockTokens.set(`mock-code-${user.userId}`, user.userId);
    });
  }

  // OAuth Authentication Flow
  async getOAuthUrl(redirectUri: string, state?: string): Promise<string> {
    const params = new URLSearchParams({
      appid: process.env.WECHAT_CORP_ID || 'mock-corp-id',
      redirect_uri: redirectUri,
      response_type: 'code',
      scope: 'snsapi_base',
      state: state || 'STATE'
    });
    
    // In mock mode, redirect to local callback with mock code
    return `${redirectUri}?code=mock-code-student001&state=${state || 'STATE'}`;
  }

  async getAccessToken(): Promise<{ access_token: string; expires_in: number }> {
    console.log('[MOCK] Getting WeChat Work access token');
    return {
      access_token: this.accessToken,
      expires_in: 7200
    };
  }

  async getUserInfoByCode(code: string): Promise<WeChatUser> {
    console.log('[MOCK] Getting user info for code:', code);
    
    // Extract userId from mock code
    const userId = this.mockTokens.get(code);
    if (!userId) {
      throw new Error('Invalid authorization code');
    }

    const user = this.mockUsers.get(userId);
    if (!user) {
      throw new Error('User not found');
    }

    return user;
  }

  async getUserInfo(userId: string): Promise<WeChatUser> {
    console.log('[MOCK] Getting user info for userId:', userId);
    
    const user = this.mockUsers.get(userId);
    if (!user) {
      // Create a new mock user on the fly
      const newUser: WeChatUser = {
        userId,
        name: `用户${userId}`,
        avatar: `https://api.dicebear.com/7.x/avataaars/svg?seed=${userId}`,
        department: ['默认部门'],
        email: `${userId}@university.edu.cn`
      };
      this.mockUsers.set(userId, newUser);
      return newUser;
    }

    return user;
  }

  // Notification Methods
  async sendTextMessage(
    toUser: string | string[],
    content: string,
    agentId?: number
  ): Promise<{ errcode: number; errmsg: string; msgid?: string }> {
    const notification = {
      touser: Array.isArray(toUser) ? toUser.join('|') : toUser,
      msgtype: 'text',
      agentid: agentId || parseInt(process.env.WECHAT_AGENT_ID || '1000001'),
      text: { content },
      timestamp: Date.now(),
      msgid: uuidv4()
    };

    this.notificationQueue.push(notification);
    console.log('[MOCK] Sending text message:', notification);

    return {
      errcode: 0,
      errmsg: 'ok',
      msgid: notification.msgid
    };
  }

  async sendMarkdownMessage(
    toUser: string | string[],
    content: string,
    agentId?: number
  ): Promise<{ errcode: number; errmsg: string; msgid?: string }> {
    const notification = {
      touser: Array.isArray(toUser) ? toUser.join('|') : toUser,
      msgtype: 'markdown',
      agentid: agentId || parseInt(process.env.WECHAT_AGENT_ID || '1000001'),
      markdown: { content },
      timestamp: Date.now(),
      msgid: uuidv4()
    };

    this.notificationQueue.push(notification);
    console.log('[MOCK] Sending markdown message:', notification);

    return {
      errcode: 0,
      errmsg: 'ok',
      msgid: notification.msgid
    };
  }

  async sendNewsMessage(
    toUser: string | string[],
    articles: Array<{
      title: string;
      description?: string;
      url: string;
      picurl?: string;
    }>,
    agentId?: number
  ): Promise<{ errcode: number; errmsg: string; msgid?: string }> {
    const notification = {
      touser: Array.isArray(toUser) ? toUser.join('|') : toUser,
      msgtype: 'news',
      agentid: agentId || parseInt(process.env.WECHAT_AGENT_ID || '1000001'),
      news: { articles },
      timestamp: Date.now(),
      msgid: uuidv4()
    };

    this.notificationQueue.push(notification);
    console.log('[MOCK] Sending news message:', notification);

    return {
      errcode: 0,
      errmsg: 'ok',
      msgid: notification.msgid
    };
  }

  // Batch Operations
  async batchSendMessages(
    messages: WeChatNotification[]
  ): Promise<Array<{ errcode: number; errmsg: string; msgid?: string }>> {
    console.log('[MOCK] Batch sending messages:', messages.length);
    
    const results = [];
    for (const message of messages) {
      // Simulate rate limiting
      await new Promise(resolve => setTimeout(resolve, 100));
      
      const result = {
        errcode: 0,
        errmsg: 'ok',
        msgid: uuidv4()
      };
      
      this.notificationQueue.push({
        ...message,
        timestamp: Date.now(),
        msgid: result.msgid
      });
      
      results.push(result);
    }
    
    return results;
  }

  // Utility Methods for Testing
  getNotificationQueue(): any[] {
    return [...this.notificationQueue];
  }

  clearNotificationQueue(): void {
    this.notificationQueue = [];
  }

  getNotificationCount(userId?: string): number {
    if (!userId) {
      return this.notificationQueue.length;
    }
    
    return this.notificationQueue.filter(n => 
      n.touser.includes(userId)
    ).length;
  }

  getMockUsers(): Map<string, WeChatUser> {
    return new Map(this.mockUsers);
  }

  addMockUser(user: WeChatUser): void {
    this.mockUsers.set(user.userId, user);
    this.mockTokens.set(`mock-code-${user.userId}`, user.userId);
  }

  // Simulate API errors for testing
  simulateError(errorType: 'rate_limit' | 'invalid_token' | 'network'): void {
    switch (errorType) {
      case 'rate_limit':
        throw {
          errcode: 45009,
          errmsg: 'api freq out of limit'
        };
      case 'invalid_token':
        throw {
          errcode: 40014,
          errmsg: 'invalid access_token'
        };
      case 'network':
        throw new Error('Network error: Connection timeout');
    }
  }
}

// Factory function for easy mock service creation
export function createWeChatWorkMock() {
  return new WeChatWorkMockService();
}
```

## 2. University SSO Mock Service

```typescript
// packages/utils/src/lib/mocks/university-sso-mock.service.ts
import { Injectable } from '@nestjs/common';
import * as jwt from 'jsonwebtoken';

export interface SSOUser {
  uid: string;
  studentId: string;
  name: string;
  email: string;
  department: string;
  role: 'student' | 'faculty' | 'staff';
  attributes: Record<string, any>;
}

export interface SAMLResponse {
  nameID: string;
  nameIDFormat: string;
  sessionIndex: string;
  attributes: Record<string, any>;
}

@Injectable()
export class UniversitySSOWMockService {
  private mockUsers = new Map<string, SSOUser>();
  private sessions = new Map<string, any>();
  private readonly SECRET = 'mock-sso-secret';

  constructor() {
    this.initializeMockUsers();
  }

  private initializeMockUsers() {
    const users: SSOUser[] = [
      {
        uid: '2021001',
        studentId: '2021001',
        name: '张小明',
        email: 'zhang.xiaoming@university.edu.cn',
        department: '计算机科学与技术',
        role: 'student',
        attributes: {
          grade: '2021',
          major: '计算机科学与技术',
          class: '计科2101',
          advisor: 'professor001'
        }
      },
      {
        uid: 'prof001',
        studentId: '',
        name: '李明教授',
        email: 'li.ming@university.edu.cn',
        department: '计算机科学与技术',
        role: 'faculty',
        attributes: {
          title: '教授',
          research_area: '人工智能,机器学习',
          office: 'A305'
        }
      }
    ];

    users.forEach(user => {
      this.mockUsers.set(user.uid, user);
      this.mockUsers.set(user.email, user);
    });
  }

  // CAS-style SSO flow
  async getLoginUrl(service: string): Promise<string> {
    const ticket = 'ST-' + Date.now() + '-' + Math.random().toString(36);
    
    // Store ticket for validation
    this.sessions.set(ticket, {
      service,
      userId: '2021001', // Default to student for demo
      timestamp: Date.now()
    });

    // In real SSO, this would redirect to SSO login page
    // In mock, directly return callback URL with ticket
    return `${service}?ticket=${ticket}`;
  }

  async validateTicket(ticket: string, service: string): Promise<SSOUser> {
    console.log('[MOCK] Validating SSO ticket:', ticket);
    
    const session = this.sessions.get(ticket);
    if (!session || session.service !== service) {
      throw new Error('Invalid ticket');
    }

    // Check ticket expiry (5 minutes)
    if (Date.now() - session.timestamp > 5 * 60 * 1000) {
      this.sessions.delete(ticket);
      throw new Error('Ticket expired');
    }

    const user = this.mockUsers.get(session.userId);
    if (!user) {
      throw new Error('User not found');
    }

    // Clean up used ticket
    this.sessions.delete(ticket);

    return user;
  }

  // SAML-style SSO flow
  async createSAMLRequest(callbackUrl: string): Promise<string> {
    const samlRequest = {
      id: '_' + Math.random().toString(36).substr(2, 9),
      issueInstant: new Date().toISOString(),
      destination: 'https://sso.university.edu.cn/saml/sso',
      issuer: 'srmp.university.edu.cn',
      callbackUrl
    };

    // In real implementation, this would be signed XML
    return Buffer.from(JSON.stringify(samlRequest)).toString('base64');
  }

  async processSAMLResponse(samlResponse: string): Promise<SAMLResponse> {
    console.log('[MOCK] Processing SAML response');
    
    // In mock, we'll just return a pre-configured response
    // In real implementation, this would validate signed XML
    return {
      nameID: '2021001',
      nameIDFormat: 'urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified',
      sessionIndex: '_' + Date.now(),
      attributes: {
        uid: '2021001',
        mail: 'zhang.xiaoming@university.edu.cn',
        displayName: '张小明',
        department: '计算机科学与技术',
        eduPersonAffiliation: 'student'
      }
    };
  }

  // JWT-based mock authentication
  async authenticateUser(username: string, password: string): Promise<{
    token: string;
    user: SSOUser;
  }> {
    console.log('[MOCK] Authenticating user:', username);
    
    // Accept any password in mock mode
    const user = this.mockUsers.get(username);
    if (!user) {
      throw new Error('Invalid credentials');
    }

    const token = jwt.sign(
      {
        uid: user.uid,
        email: user.email,
        role: user.role
      },
      this.SECRET,
      { expiresIn: '24h' }
    );

    return { token, user };
  }

  async verifyToken(token: string): Promise<any> {
    try {
      return jwt.verify(token, this.SECRET);
    } catch (error) {
      throw new Error('Invalid token');
    }
  }

  // Utility methods
  async getUserAttributes(uid: string): Promise<Record<string, any>> {
    const user = this.mockUsers.get(uid);
    if (!user) {
      throw new Error('User not found');
    }

    return {
      ...user.attributes,
      uid: user.uid,
      name: user.name,
      email: user.email,
      department: user.department,
      role: user.role
    };
  }

  async searchUsers(query: string): Promise<SSOUser[]> {
    const results: SSOUser[] = [];
    
    this.mockUsers.forEach(user => {
      if (
        user.name.includes(query) ||
        user.email.includes(query) ||
        user.studentId.includes(query)
      ) {
        results.push(user);
      }
    });

    return results;
  }

  // Add mock user for testing
  addMockUser(user: SSOUser): void {
    this.mockUsers.set(user.uid, user);
    if (user.email) {
      this.mockUsers.set(user.email, user);
    }
  }

  // Simulate SSO errors
  simulateError(type: 'timeout' | 'invalid_credentials' | 'service_error'): void {
    switch (type) {
      case 'timeout':
        throw new Error('SSO service timeout');
      case 'invalid_credentials':
        throw new Error('Invalid username or password');
      case 'service_error':
        throw new Error('SSO service temporarily unavailable');
    }
  }
}
```

## 3. Email/SMS Service Mock

```typescript
// packages/utils/src/lib/mocks/communication-mock.service.ts
import { Injectable } from '@nestjs/common';

export interface EmailMessage {
  to: string | string[];
  subject: string;
  html?: string;
  text?: string;
  cc?: string[];
  bcc?: string[];
  attachments?: Array<{
    filename: string;
    content: Buffer | string;
  }>;
}

export interface SMSMessage {
  to: string;
  content: string;
  templateId?: string;
  params?: Record<string, string>;
}

@Injectable()
export class CommunicationMockService {
  private emailQueue: any[] = [];
  private smsQueue: any[] = [];
  private emailTemplates = new Map<string, string>();
  private smsTemplates = new Map<string, string>();

  constructor() {
    this.initializeTemplates();
  }

  private initializeTemplates() {
    // Email templates
    this.emailTemplates.set('welcome', `
      <h1>欢迎加入科研管理平台</h1>
      <p>尊敬的 {{name}}，</p>
      <p>您的账号已创建成功。</p>
      <p>登录邮箱：{{email}}</p>
      <p>请点击下方链接完成邮箱验证：</p>
      <a href="{{verifyLink}}">验证邮箱</a>
    `);

    this.emailTemplates.set('project-accepted', `
      <h1>项目申请已通过</h1>
      <p>恭喜！您申请的项目 "{{projectTitle}}" 已被接受。</p>
      <p>指导教授：{{professorName}}</p>
      <p>开始日期：{{startDate}}</p>
    `);

    // SMS templates
    this.smsTemplates.set('verification', 
      '【科研平台】您的验证码是{{code}}，5分钟内有效。'
    );

    this.smsTemplates.set('deadline-reminder',
      '【科研平台】提醒：{{projectName}}的{{milestone}}截止日期为{{deadline}}，请及时提交。'
    );
  }

  // Email methods
  async sendEmail(message: EmailMessage): Promise<{
    messageId: string;
    accepted: string[];
    rejected: string[];
  }> {
    console.log('[MOCK] Sending email:', {
      to: message.to,
      subject: message.subject
    });

    const messageId = `<${Date.now()}@mock.srmp.edu.cn>`;
    const recipients = Array.isArray(message.to) ? message.to : [message.to];

    this.emailQueue.push({
      ...message,
      messageId,
      timestamp: new Date(),
      status: 'sent'
    });

    return {
      messageId,
      accepted: recipients,
      rejected: []
    };
  }

  async sendTemplateEmail(
    to: string | string[],
    templateId: string,
    params: Record<string, any>
  ): Promise<any> {
    const template = this.emailTemplates.get(templateId);
    if (!template) {
      throw new Error(`Template ${templateId} not found`);
    }

    let html = template;
    Object.entries(params).forEach(([key, value]) => {
      html = html.replace(new RegExp(`{{${key}}}`, 'g'), value);
    });

    return this.sendEmail({
      to,
      subject: params.subject || `Email from SRMP`,
      html
    });
  }

  // SMS methods
  async sendSMS(message: SMSMessage): Promise<{
    messageId: string;
    status: 'sent' | 'failed';
    errorMessage?: string;
  }> {
    console.log('[MOCK] Sending SMS:', {
      to: message.to,
      content: message.content.substring(0, 20) + '...'
    });

    // Validate phone number
    if (!message.to.match(/^1[3-9]\d{9}$/)) {
      return {
        messageId: '',
        status: 'failed',
        errorMessage: 'Invalid phone number'
      };
    }

    const messageId = 'SMS' + Date.now();

    this.smsQueue.push({
      ...message,
      messageId,
      timestamp: new Date(),
      status: 'sent'
    });

    return {
      messageId,
      status: 'sent'
    };
  }

  async sendTemplateSMS(
    to: string,
    templateId: string,
    params: Record<string, string>
  ): Promise<any> {
    const template = this.smsTemplates.get(templateId);
    if (!template) {
      throw new Error(`SMS template ${templateId} not found`);
    }

    let content = template;
    Object.entries(params).forEach(([key, value]) => {
      content = content.replace(new RegExp(`{{${key}}}`, 'g'), value);
    });

    return this.sendSMS({ to, content, templateId, params });
  }

  // Verification code
  async sendVerificationCode(
    to: string,
    type: 'email' | 'sms'
  ): Promise<{ code: string; expiresAt: Date }> {
    const code = Math.floor(100000 + Math.random() * 900000).toString();
    const expiresAt = new Date(Date.now() + 5 * 60 * 1000); // 5 minutes

    if (type === 'email') {
      await this.sendTemplateEmail(to, 'verification', {
        code,
        subject: '邮箱验证码'
      });
    } else {
      await this.sendTemplateSMS(to, 'verification', { code });
    }

    return { code, expiresAt };
  }

  // Batch operations
  async sendBatchEmails(
    messages: EmailMessage[]
  ): Promise<Array<{ to: string; success: boolean; messageId?: string }>> {
    const results = [];

    for (const message of messages) {
      try {
        const result = await this.sendEmail(message);
        results.push({
          to: Array.isArray(message.to) ? message.to[0] : message.to,
          success: true,
          messageId: result.messageId
        });
      } catch (error) {
        results.push({
          to: Array.isArray(message.to) ? message.to[0] : message.to,
          success: false
        });
      }

      // Simulate rate limiting
      await new Promise(resolve => setTimeout(resolve, 100));
    }

    return results;
  }

  // Utility methods
  getEmailQueue(): any[] {
    return [...this.emailQueue];
  }

  getSMSQueue(): any[] {
    return [...this.smsQueue];
  }

  clearQueues(): void {
    this.emailQueue = [];
    this.smsQueue = [];
  }

  getMessageStatus(messageId: string): any {
    const email = this.emailQueue.find(m => m.messageId === messageId);
    if (email) return { type: 'email', ...email };

    const sms = this.smsQueue.find(m => m.messageId === messageId);
    if (sms) return { type: 'sms', ...sms };

    return null;
  }
}
```

## 4. Alibaba Cloud Services Mock

```typescript
// packages/utils/src/lib/mocks/alibaba-cloud-mock.service.ts
import { Injectable } from '@nestjs/common';
import * as crypto from 'crypto';

export interface OSSObject {
  key: string;
  size: number;
  lastModified: Date;
  etag: string;
  url: string;
}

@Injectable()
export class AlibabaCloudMockService {
  private ossObjects = new Map<string, OSSObject>();
  private kmsSecrets = new Map<string, string>();
  
  // OSS (Object Storage Service) Mock
  async uploadFile(
    bucket: string,
    key: string,
    buffer: Buffer,
    options?: {
      contentType?: string;
      metadata?: Record<string, string>;
    }
  ): Promise<OSSObject> {
    console.log(`[MOCK] Uploading to OSS: ${bucket}/${key}`);
    
    const object: OSSObject = {
      key,
      size: buffer.length,
      lastModified: new Date(),
      etag: crypto.createHash('md5').update(buffer).digest('hex'),
      url: `https://${bucket}.oss-cn-beijing.aliyuncs.com/${key}`
    };

    this.ossObjects.set(`${bucket}/${key}`, object);
    
    return object;
  }

  async getSignedUrl(
    bucket: string,
    key: string,
    expires: number = 3600
  ): Promise<string> {
    const expiresAt = Date.now() + expires * 1000;
    const signature = crypto
      .createHash('sha256')
      .update(`${bucket}/${key}/${expiresAt}`)
      .digest('hex');
    
    return `https://${bucket}.oss-cn-beijing.aliyuncs.com/${key}?signature=${signature}&expires=${expiresAt}`;
  }

  async deleteFile(bucket: string, key: string): Promise<void> {
    console.log(`[MOCK] Deleting from OSS: ${bucket}/${key}`);
    this.ossObjects.delete(`${bucket}/${key}`);
  }

  async listFiles(
    bucket: string,
    prefix?: string
  ): Promise<OSSObject[]> {
    const results: OSSObject[] = [];
    
    this.ossObjects.forEach((object, fullKey) => {
      if (fullKey.startsWith(`${bucket}/`)) {
        const key = fullKey.substring(bucket.length + 1);
        if (!prefix || key.startsWith(prefix)) {
          results.push(object);
        }
      }
    });
    
    return results;
  }

  // KMS (Key Management Service) Mock
  async encrypt(
    keyId: string,
    plaintext: string
  ): Promise<{ ciphertextBlob: string }> {
    console.log(`[MOCK] KMS Encrypt with key: ${keyId}`);
    
    const cipher = crypto.createCipher('aes-256-cbc', keyId);
    const encrypted = Buffer.concat([
      cipher.update(plaintext, 'utf8'),
      cipher.final()
    ]);
    
    return {
      ciphertextBlob: encrypted.toString('base64')
    };
  }

  async decrypt(
    ciphertextBlob: string
  ): Promise<{ plaintext: string }> {
    console.log(`[MOCK] KMS Decrypt`);
    
    // In mock, use a fixed key
    const keyId = 'mock-kms-key';
    const decipher = crypto.createDecipher('aes-256-cbc', keyId);
    const decrypted = Buffer.concat([
      decipher.update(Buffer.from(ciphertextBlob, 'base64')),
      decipher.final()
    ]);
    
    return {
      plaintext: decrypted.toString('utf8')
    };
  }

  async createSecret(
    name: string,
    value: string,
    description?: string
  ): Promise<{ arn: string; name: string }> {
    console.log(`[MOCK] Creating secret: ${name}`);
    
    this.kmsSecrets.set(name, value);
    
    return {
      arn: `acs:kms:cn-beijing:mock:secret/${name}`,
      name
    };
  }

  async getSecretValue(name: string): Promise<{ value: string }> {
    const value = this.kmsSecrets.get(name);
    if (!value) {
      throw new Error(`Secret ${name} not found`);
    }
    
    return { value };
  }

  // SLS (Simple Log Service) Mock
  async writeLog(
    project: string,
    logstore: string,
    logs: Array<{
      timestamp: number;
      content: Record<string, any>;
    }>
  ): Promise<void> {
    console.log(`[MOCK] Writing ${logs.length} logs to ${project}/${logstore}`);
    
    logs.forEach(log => {
      console.log(`[SLS] ${new Date(log.timestamp * 1000).toISOString()}`, log.content);
    });
  }

  // Function Compute Mock
  async invokeFunction(
    serviceName: string,
    functionName: string,
    payload: any
  ): Promise<any> {
    console.log(`[MOCK] Invoking function: ${serviceName}/${functionName}`);
    
    // Mock some common functions
    if (functionName === 'image-resize') {
      return {
        success: true,
        resizedUrl: `https://mock.oss-cn-beijing.aliyuncs.com/resized-${Date.now()}.jpg`
      };
    }
    
    if (functionName === 'pdf-generate') {
      return {
        success: true,
        pdfUrl: `https://mock.oss-cn-beijing.aliyuncs.com/report-${Date.now()}.pdf`
      };
    }
    
    return { success: true, result: 'Function executed' };
  }

  // Monitoring Mock
  async putMetric(
    namespace: string,
    metricName: string,
    value: number,
    unit: string = 'Count'
  ): Promise<void> {
    console.log(`[MOCK] Metric: ${namespace}/${metricName} = ${value} ${unit}`);
  }

  // Utility methods
  clearOSSObjects(): void {
    this.ossObjects.clear();
  }

  getOSSObjectCount(): number {
    return this.ossObjects.size;
  }

  clearSecrets(): void {
    this.kmsSecrets.clear();
  }
}
```

## 5. Mock Service Module

```typescript
// packages/utils/src/lib/mocks/mock-services.module.ts
import { Module, DynamicModule, Global } from '@nestjs/common';
import { WeChatWorkMockService } from './wechat-work-mock.service';
import { UniversitySSOWMockService } from './university-sso-mock.service';
import { CommunicationMockService } from './communication-mock.service';
import { AlibabaCloudMockService } from './alibaba-cloud-mock.service';

export interface MockServicesConfig {
  enableMocks: boolean;
  mockDelay?: number; // Simulate network delay
  errorRate?: number; // Simulate random errors (0-1)
}

@Global()
@Module({})
export class MockServicesModule {
  static forRoot(config: MockServicesConfig): DynamicModule {
    const providers = config.enableMocks
      ? [
          {
            provide: 'WECHAT_SERVICE',
            useClass: WeChatWorkMockService,
          },
          {
            provide: 'SSO_SERVICE',
            useClass: UniversitySSOWMockService,
          },
          {
            provide: 'COMM_SERVICE',
            useClass: CommunicationMockService,
          },
          {
            provide: 'CLOUD_SERVICE',
            useClass: AlibabaCloudMockService,
          },
          {
            provide: 'MOCK_CONFIG',
            useValue: config,
          },
        ]
      : [];

    return {
      module: MockServicesModule,
      providers,
      exports: providers,
    };
  }
}
```

## 6. Mock Service Interceptor

```typescript
// packages/utils/src/lib/mocks/mock.interceptor.ts
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
  Inject,
} from '@nestjs/common';
import { Observable, of, throwError } from 'rxjs';
import { delay, mergeMap } from 'rxjs/operators';

@Injectable()
export class MockInterceptor implements NestInterceptor {
  constructor(
    @Inject('MOCK_CONFIG') private config: any
  ) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    // Only apply in development/test with mocks enabled
    if (!this.config.enableMocks) {
      return next.handle();
    }

    // Simulate network delay
    const delayTime = this.config.mockDelay || 0;

    // Simulate random errors
    const shouldError = Math.random() < (this.config.errorRate || 0);
    
    if (shouldError) {
      return of(null).pipe(
        delay(delayTime),
        mergeMap(() => throwError(new Error('Mock service error')))
      );
    }

    return next.handle().pipe(delay(delayTime));
  }
}
```

## 7. Usage Example

```typescript
// apps/api-gateway/src/app.module.ts
import { Module } from '@nestjs/common';
import { MockServicesModule } from '@srmp/utils';

@Module({
  imports: [
    MockServicesModule.forRoot({
      enableMocks: process.env.NODE_ENV !== 'production',
      mockDelay: 200, // 200ms delay
      errorRate: 0.05, // 5% error rate
    }),
    // ... other modules
  ],
})
export class AppModule {}

// In a service
import { Injectable, Inject } from '@nestjs/common';

@Injectable()
export class NotificationService {
  constructor(
    @Inject('WECHAT_SERVICE') private wechat: any,
    @Inject('COMM_SERVICE') private comm: any,
  ) {}

  async notifyUser(userId: string, message: string) {
    try {
      // Try WeChat first
      await this.wechat.sendTextMessage(userId, message);
    } catch (error) {
      // Fallback to email
      const user = await this.userService.findById(userId);
      await this.comm.sendEmail({
        to: user.email,
        subject: 'Notification from SRMP',
        text: message,
      });
    }
  }
}
```

## 8. Testing with Mocks

```typescript
// apps/api-gateway/src/auth/auth.service.spec.ts
import { Test } from '@nestjs/testing';
import { AuthService } from './auth.service';
import { WeChatWorkMockService } from '@srmp/utils';

describe('AuthService', () => {
  let service: AuthService;
  let wechatMock: WeChatWorkMockService;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        AuthService,
        {
          provide: 'WECHAT_SERVICE',
          useClass: WeChatWorkMockService,
        },
      ],
    }).compile();

    service = module.get<AuthService>(AuthService);
    wechatMock = module.get('WECHAT_SERVICE');
  });

  it('should authenticate with WeChat', async () => {
    const code = 'mock-code-student001';
    const result = await service.wechatLogin(code);
    
    expect(result).toHaveProperty('token');
    expect(result.user.userId).toBe('student001');
  });

  it('should handle WeChat errors gracefully', async () => {
    jest.spyOn(wechatMock, 'getUserInfoByCode')
      .mockRejectedValueOnce(new Error('Invalid code'));
    
    await expect(service.wechatLogin('invalid'))
      .rejects.toThrow('Authentication failed');
  });
});
```

These mock services provide complete offline development capability with realistic behavior, error simulation, and testing utilities. They can be easily swapped with real implementations when moving to production.