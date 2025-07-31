# Risk Monitoring Dashboard Template - SRMP

## Dashboard Overview

This risk monitoring dashboard provides real-time visibility into the top 5 risks identified during the PO validation, with automated monitoring and alerting capabilities.

## 1. Dashboard Layout

```typescript
// apps/web/src/pages/admin/RiskDashboard.tsx
import React from 'react';
import { Row, Col, Card, Progress, Alert, Statistic, Badge } from 'antd';
import { 
  WarningOutlined, 
  CheckCircleOutlined, 
  CloseCircleOutlined,
  SyncOutlined 
} from '@ant-design/icons';

export const RiskDashboard: React.FC = () => {
  const risks = useRiskMonitoring();
  
  return (
    <div className="risk-dashboard">
      <h1>SRMP Risk Monitoring Dashboard</h1>
      
      {/* Overall Risk Score */}
      <Card className="mb-4">
        <Row gutter={[16, 16]}>
          <Col span={6}>
            <Statistic
              title="Overall Risk Level"
              value={risks.overallScore}
              suffix="/ 100"
              valueStyle={{ color: getRiskColor(risks.overallScore) }}
            />
          </Col>
          <Col span={6}>
            <Statistic
              title="Active Risks"
              value={risks.activeCount}
              prefix={<WarningOutlined />}
              valueStyle={{ color: '#faad14' }}
            />
          </Col>
          <Col span={6}>
            <Statistic
              title="Mitigated Risks"
              value={risks.mitigatedCount}
              prefix={<CheckCircleOutlined />}
              valueStyle={{ color: '#52c41a' }}
            />
          </Col>
          <Col span={6}>
            <Statistic
              title="Last Updated"
              value={risks.lastUpdated}
              prefix={<SyncOutlined spin={risks.isUpdating} />}
            />
          </Col>
        </Row>
      </Card>

      {/* Risk Cards */}
      <Row gutter={[16, 16]}>
        {risks.items.map(risk => (
          <Col span={24} key={risk.id}>
            <RiskCard risk={risk} />
          </Col>
        ))}
      </Row>
    </div>
  );
};
```

## 2. Risk Monitoring Components

### Risk Card Component

```typescript
// components/admin/RiskCard.tsx
interface RiskCardProps {
  risk: RiskItem;
}

const RiskCard: React.FC<RiskCardProps> = ({ risk }) => {
  const [expanded, setExpanded] = useState(false);
  
  return (
    <Card
      title={
        <Space>
          <Badge status={getRiskStatus(risk.severity)} />
          <span>{risk.title}</span>
          <Tag color={getRiskTagColor(risk.severity)}>
            {risk.severity}
          </Tag>
        </Space>
      }
      extra={
        <Space>
          <Tooltip title="Risk Score">
            <Progress
              type="circle"
              percent={risk.score}
              width={50}
              strokeColor={getRiskColor(risk.score)}
            />
          </Tooltip>
          <Button
            type="link"
            onClick={() => setExpanded(!expanded)}
          >
            {expanded ? 'Collapse' : 'Details'}
          </Button>
        </Space>
      }
    >
      {/* Risk Summary */}
      <Row gutter={[16, 16]}>
        <Col span={8}>
          <Statistic
            title="Impact"
            value={risk.impact}
            suffix="/ 10"
          />
        </Col>
        <Col span={8}>
          <Statistic
            title="Likelihood"
            value={risk.likelihood}
            suffix="%"
          />
        </Col>
        <Col span={8}>
          <Statistic
            title="Detection Time"
            value={risk.detectionTime}
            suffix="min"
          />
        </Col>
      </Row>

      {/* Mitigation Progress */}
      <div className="mt-4">
        <h4>Mitigation Progress</h4>
        <Progress
          percent={risk.mitigationProgress}
          steps={risk.mitigationSteps}
          strokeColor="#1890ff"
        />
      </div>

      {/* Expanded Details */}
      {expanded && (
        <div className="mt-4">
          <Descriptions bordered column={1} size="small">
            <Descriptions.Item label="Description">
              {risk.description}
            </Descriptions.Item>
            <Descriptions.Item label="Current Status">
              {risk.currentStatus}
            </Descriptions.Item>
            <Descriptions.Item label="Mitigation Strategy">
              {risk.mitigationStrategy}
            </Descriptions.Item>
            <Descriptions.Item label="Owner">
              {risk.owner}
            </Descriptions.Item>
            <Descriptions.Item label="Next Review">
              {risk.nextReview}
            </Descriptions.Item>
          </Descriptions>

          {/* Action Buttons */}
          <Space className="mt-3">
            <Button type="primary" icon={<CheckOutlined />}>
              Update Status
            </Button>
            <Button icon={<FileTextOutlined />}>
              View History
            </Button>
            <Button icon={<BellOutlined />}>
              Configure Alerts
            </Button>
          </Space>
        </div>
      )}
    </Card>
  );
};
```

## 3. Risk Monitoring Service

```typescript
// services/risk-monitoring.service.ts
import { Injectable } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';
import { EventEmitter2 } from '@nestjs/event-emitter';

export interface RiskMetrics {
  wechatIntegration: {
    registrationStatus: 'pending' | 'in-progress' | 'completed' | 'failed';
    daysRemaining: number;
    apiHealth: 'healthy' | 'degraded' | 'down';
    rateLimitUsage: number;
    lastError?: string;
  };
  aiMatching: {
    complexity: 'simple' | 'moderate' | 'complex';
    dataAvailable: boolean;
    mlInfrastructure: boolean;
    fallbackActive: boolean;
    accuracy?: number;
  };
  database: {
    migrationsPending: number;
    schemaVersion: string;
    backupStatus: 'current' | 'outdated' | 'failed';
    replicationLag: number;
  };
  infrastructure: {
    servicesHealth: Record<string, 'up' | 'down'>;
    resourceUtilization: {
      cpu: number;
      memory: number;
      disk: number;
    };
    alertsActive: number;
  };
  thirdParty: {
    services: Array<{
      name: string;
      status: 'operational' | 'degraded' | 'down';
      lastCheck: Date;
      uptime: number;
    }>;
  };
}

@Injectable()
export class RiskMonitoringService {
  constructor(
    private eventEmitter: EventEmitter2,
    private wechatService: WeChatService,
    private dbService: DatabaseService,
    private cloudService: CloudMonitoringService,
  ) {}

  // Monitor WeChat Integration Risk
  @Cron(CronExpression.EVERY_HOUR)
  async monitorWeChatIntegration() {
    const metrics: RiskMetrics['wechatIntegration'] = {
      registrationStatus: await this.checkRegistrationStatus(),
      daysRemaining: await this.calculateDaysRemaining(),
      apiHealth: await this.wechatService.healthCheck(),
      rateLimitUsage: await this.wechatService.getRateLimitUsage(),
    };

    // Calculate risk score
    let riskScore = 0;
    
    if (metrics.registrationStatus === 'pending') riskScore += 30;
    if (metrics.daysRemaining < 7) riskScore += 20;
    if (metrics.apiHealth !== 'healthy') riskScore += 25;
    if (metrics.rateLimitUsage > 80) riskScore += 25;

    if (riskScore > 50) {
      this.eventEmitter.emit('risk.alert', {
        type: 'wechat-integration',
        severity: riskScore > 75 ? 'high' : 'medium',
        metrics,
      });
    }

    return { score: riskScore, metrics };
  }

  // Monitor AI Matching Complexity
  @Cron(CronExpression.EVERY_DAY)
  async monitorAIMatching() {
    const metrics: RiskMetrics['aiMatching'] = {
      complexity: await this.assessComplexity(),
      dataAvailable: await this.checkTrainingData(),
      mlInfrastructure: await this.checkMLInfra(),
      fallbackActive: await this.isFallbackActive(),
      accuracy: await this.measureAccuracy(),
    };

    let riskScore = 0;
    
    if (metrics.complexity === 'complex' && !metrics.mlInfrastructure) {
      riskScore += 40;
    }
    if (!metrics.dataAvailable) riskScore += 30;
    if (metrics.accuracy && metrics.accuracy < 0.7) riskScore += 30;

    return { score: riskScore, metrics };
  }

  // Monitor Database Schema Evolution
  @Cron(CronExpression.EVERY_30_MINUTES)
  async monitorDatabase() {
    const metrics: RiskMetrics['database'] = {
      migrationsPending: await this.dbService.getPendingMigrations(),
      schemaVersion: await this.dbService.getSchemaVersion(),
      backupStatus: await this.dbService.getBackupStatus(),
      replicationLag: await this.dbService.getReplicationLag(),
    };

    let riskScore = 0;
    
    if (metrics.migrationsPending > 5) riskScore += 30;
    if (metrics.backupStatus !== 'current') riskScore += 40;
    if (metrics.replicationLag > 1000) riskScore += 30;

    return { score: riskScore, metrics };
  }

  // Monitor Infrastructure Services
  @Cron(CronExpression.EVERY_5_MINUTES)
  async monitorInfrastructure() {
    const services = ['redis', 'elasticsearch', 'rocketmq', 'postgres'];
    const health: Record<string, 'up' | 'down'> = {};
    
    for (const service of services) {
      health[service] = await this.checkServiceHealth(service);
    }

    const metrics: RiskMetrics['infrastructure'] = {
      servicesHealth: health,
      resourceUtilization: await this.cloudService.getResourceMetrics(),
      alertsActive: await this.cloudService.getActiveAlerts(),
    };

    const downServices = Object.values(health).filter(s => s === 'down').length;
    let riskScore = downServices * 25;
    
    const { cpu, memory, disk } = metrics.resourceUtilization;
    if (cpu > 80) riskScore += 15;
    if (memory > 85) riskScore += 15;
    if (disk > 90) riskScore += 20;

    return { score: riskScore, metrics };
  }

  // Monitor Third-Party Services
  @Cron(CronExpression.EVERY_10_MINUTES)
  async monitorThirdPartyServices() {
    const services = [
      { name: 'WeChat Work API', endpoint: 'https://qyapi.weixin.qq.com/cgi-bin/gettoken' },
      { name: 'University SSO', endpoint: 'https://sso.university.edu.cn/health' },
      { name: 'Alibaba OSS', endpoint: 'https://oss-cn-beijing.aliyuncs.com' },
    ];

    const results = await Promise.all(
      services.map(async (service) => ({
        name: service.name,
        status: await this.checkEndpoint(service.endpoint),
        lastCheck: new Date(),
        uptime: await this.getUptime(service.name),
      }))
    );

    const downServices = results.filter(s => s.status === 'down').length;
    const degradedServices = results.filter(s => s.status === 'degraded').length;
    
    const riskScore = (downServices * 30) + (degradedServices * 15);

    return {
      score: riskScore,
      metrics: { services: results },
    };
  }

  // Aggregate all risks
  async calculateOverallRisk(): Promise<{
    overall: number;
    breakdown: Record<string, number>;
    recommendations: string[];
  }> {
    const risks = await Promise.all([
      this.monitorWeChatIntegration(),
      this.monitorAIMatching(),
      this.monitorDatabase(),
      this.monitorInfrastructure(),
      this.monitorThirdPartyServices(),
    ]);

    const breakdown = {
      wechatIntegration: risks[0].score,
      aiMatching: risks[1].score,
      database: risks[2].score,
      infrastructure: risks[3].score,
      thirdParty: risks[4].score,
    };

    const overall = Math.min(
      100,
      Object.values(breakdown).reduce((sum, score) => sum + score * 0.2, 0)
    );

    const recommendations = this.generateRecommendations(breakdown);

    return { overall, breakdown, recommendations };
  }

  private generateRecommendations(breakdown: Record<string, number>): string[] {
    const recommendations: string[] = [];

    if (breakdown.wechatIntegration > 50) {
      recommendations.push('Urgent: Complete WeChat Work registration immediately');
      recommendations.push('Implement comprehensive WeChat API mocking');
    }

    if (breakdown.aiMatching > 40) {
      recommendations.push('Simplify matching algorithm to rule-based for MVP');
      recommendations.push('Begin collecting training data during MVP phase');
    }

    if (breakdown.database > 40) {
      recommendations.push('Schedule database backup verification');
      recommendations.push('Review and consolidate pending migrations');
    }

    if (breakdown.infrastructure > 50) {
      recommendations.push('Investigate service health issues immediately');
      recommendations.push('Scale resources if utilization is high');
    }

    if (breakdown.thirdParty > 30) {
      recommendations.push('Activate fallback mechanisms for down services');
      recommendations.push('Review SLAs with third-party providers');
    }

    return recommendations;
  }
}
```

## 4. Real-time Risk Alerts

```typescript
// services/risk-alert.service.ts
import { Injectable } from '@nestjs/common';
import { OnEvent } from '@nestjs/event-emitter';

export interface RiskAlert {
  id: string;
  timestamp: Date;
  type: string;
  severity: 'low' | 'medium' | 'high' | 'critical';
  title: string;
  description: string;
  metrics: any;
  actions: string[];
}

@Injectable()
export class RiskAlertService {
  private alerts: RiskAlert[] = [];
  private subscribers: Set<(alert: RiskAlert) => void> = new Set();

  @OnEvent('risk.alert')
  async handleRiskAlert(payload: any) {
    const alert: RiskAlert = {
      id: uuidv4(),
      timestamp: new Date(),
      type: payload.type,
      severity: payload.severity,
      title: this.generateTitle(payload),
      description: this.generateDescription(payload),
      metrics: payload.metrics,
      actions: this.generateActions(payload),
    };

    this.alerts.push(alert);
    this.notifySubscribers(alert);

    // Send critical alerts immediately
    if (alert.severity === 'critical') {
      await this.sendImmediateAlert(alert);
    }
  }

  private generateTitle(payload: any): string {
    const titles: Record<string, string> = {
      'wechat-integration': 'WeChat Integration Risk Detected',
      'ai-matching': 'AI Matching System Risk',
      'database': 'Database Risk Alert',
      'infrastructure': 'Infrastructure Service Risk',
      'third-party': 'Third-Party Service Risk',
    };

    return titles[payload.type] || 'Unknown Risk Detected';
  }

  private generateActions(payload: any): string[] {
    const actionMap: Record<string, string[]> = {
      'wechat-integration': [
        'Check WeChat Work registration status',
        'Verify API credentials',
        'Review rate limit usage',
        'Test fallback email system',
      ],
      'ai-matching': [
        'Review matching algorithm complexity',
        'Verify rule-based fallback is active',
        'Check training data availability',
        'Monitor matching accuracy',
      ],
      'database': [
        'Run pending migrations in staging',
        'Verify backup integrity',
        'Check replication status',
        'Review schema changes',
      ],
    };

    return actionMap[payload.type] || ['Review system logs', 'Contact DevOps team'];
  }

  async sendImmediateAlert(alert: RiskAlert) {
    // Send to multiple channels
    await Promise.all([
      this.sendWeChatAlert(alert),
      this.sendEmailAlert(alert),
      this.createIncident(alert),
    ]);
  }

  subscribe(callback: (alert: RiskAlert) => void) {
    this.subscribers.add(callback);
    return () => this.subscribers.delete(callback);
  }

  private notifySubscribers(alert: RiskAlert) {
    this.subscribers.forEach(callback => callback(alert));
  }

  getRecentAlerts(limit: number = 10): RiskAlert[] {
    return this.alerts
      .sort((a, b) => b.timestamp.getTime() - a.timestamp.getTime())
      .slice(0, limit);
  }

  getAlertsByType(type: string): RiskAlert[] {
    return this.alerts.filter(alert => alert.type === type);
  }

  acknowledgeAlert(alertId: string, userId: string) {
    const alert = this.alerts.find(a => a.id === alertId);
    if (alert) {
      alert.acknowledged = true;
      alert.acknowledgedBy = userId;
      alert.acknowledgedAt = new Date();
    }
  }
}
```

## 5. Risk Dashboard WebSocket Updates

```typescript
// gateways/risk-monitoring.gateway.ts
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  OnGatewayConnection,
  OnGatewayDisconnect,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
import { UseGuards } from '@nestjs/common';

@WebSocketGateway({
  namespace: 'risk-monitoring',
  cors: {
    origin: process.env.FRONTEND_URL,
    credentials: true,
  },
})
@UseGuards(WsAuthGuard)
export class RiskMonitoringGateway implements OnGatewayConnection, OnGatewayDisconnect {
  @WebSocketServer()
  server: Server;

  constructor(
    private riskService: RiskMonitoringService,
    private alertService: RiskAlertService,
  ) {
    // Send updates every 30 seconds
    setInterval(() => this.broadcastRiskUpdate(), 30000);
    
    // Subscribe to alerts
    this.alertService.subscribe((alert) => {
      this.server.emit('risk-alert', alert);
    });
  }

  async handleConnection(client: Socket) {
    console.log(`Risk monitoring client connected: ${client.id}`);
    
    // Send initial data
    const currentRisk = await this.riskService.calculateOverallRisk();
    client.emit('risk-update', currentRisk);
    
    const recentAlerts = this.alertService.getRecentAlerts();
    client.emit('recent-alerts', recentAlerts);
  }

  handleDisconnect(client: Socket) {
    console.log(`Risk monitoring client disconnected: ${client.id}`);
  }

  @SubscribeMessage('request-update')
  async handleUpdateRequest(client: Socket) {
    const currentRisk = await this.riskService.calculateOverallRisk();
    client.emit('risk-update', currentRisk);
  }

  @SubscribeMessage('acknowledge-alert')
  async handleAcknowledgeAlert(
    client: Socket,
    payload: { alertId: string; userId: string }
  ) {
    this.alertService.acknowledgeAlert(payload.alertId, payload.userId);
    this.server.emit('alert-acknowledged', payload);
  }

  private async broadcastRiskUpdate() {
    const currentRisk = await this.riskService.calculateOverallRisk();
    this.server.emit('risk-update', currentRisk);
  }
}
```

## 6. Frontend Risk Monitoring Hook

```typescript
// hooks/useRiskMonitoring.ts
import { useState, useEffect, useCallback } from 'react';
import { io, Socket } from 'socket.io-client';
import { notification } from 'antd';

export interface RiskData {
  overall: number;
  breakdown: Record<string, number>;
  recommendations: string[];
  alerts: RiskAlert[];
  lastUpdated: string;
  isUpdating: boolean;
}

export const useRiskMonitoring = () => {
  const [socket, setSocket] = useState<Socket | null>(null);
  const [riskData, setRiskData] = useState<RiskData>({
    overall: 0,
    breakdown: {},
    recommendations: [],
    alerts: [],
    lastUpdated: new Date().toLocaleTimeString(),
    isUpdating: false,
  });

  useEffect(() => {
    const newSocket = io(`${process.env.VITE_WEBSOCKET_URL}/risk-monitoring`, {
      auth: {
        token: getAuthToken(),
      },
    });

    newSocket.on('connect', () => {
      console.log('Connected to risk monitoring');
    });

    newSocket.on('risk-update', (data) => {
      setRiskData(prev => ({
        ...prev,
        ...data,
        lastUpdated: new Date().toLocaleTimeString(),
        isUpdating: false,
      }));
    });

    newSocket.on('risk-alert', (alert: RiskAlert) => {
      // Show notification for new alerts
      notification.warning({
        message: alert.title,
        description: alert.description,
        duration: 0, // Don't auto-close critical alerts
        btn: (
          <Button
            size="small"
            type="primary"
            onClick={() => acknowledgeAlert(alert.id)}
          >
            Acknowledge
          </Button>
        ),
      });

      // Add to alerts list
      setRiskData(prev => ({
        ...prev,
        alerts: [alert, ...prev.alerts].slice(0, 50), // Keep last 50
      }));
    });

    newSocket.on('recent-alerts', (alerts: RiskAlert[]) => {
      setRiskData(prev => ({
        ...prev,
        alerts,
      }));
    });

    setSocket(newSocket);

    return () => {
      newSocket.close();
    };
  }, []);

  const requestUpdate = useCallback(() => {
    if (socket) {
      setRiskData(prev => ({ ...prev, isUpdating: true }));
      socket.emit('request-update');
    }
  }, [socket]);

  const acknowledgeAlert = useCallback((alertId: string) => {
    if (socket) {
      const userId = getCurrentUserId();
      socket.emit('acknowledge-alert', { alertId, userId });
    }
  }, [socket]);

  return {
    ...riskData,
    requestUpdate,
    acknowledgeAlert,
    items: transformToRiskItems(riskData),
    activeCount: riskData.alerts.filter(a => !a.acknowledged).length,
    mitigatedCount: riskData.alerts.filter(a => a.acknowledged).length,
  };
};

// Transform risk data to display format
function transformToRiskItems(data: RiskData): RiskItem[] {
  return Object.entries(data.breakdown).map(([key, score]) => ({
    id: key,
    title: getRiskTitle(key),
    severity: getRiskSeverity(score),
    score,
    impact: getImpact(key),
    likelihood: getLikelihood(key, score),
    detectionTime: getDetectionTime(key),
    mitigationProgress: getMitigationProgress(key),
    mitigationSteps: getMitigationSteps(key),
    description: getRiskDescription(key),
    currentStatus: getCurrentStatus(key, score),
    mitigationStrategy: getMitigationStrategy(key),
    owner: getRiskOwner(key),
    nextReview: getNextReview(key),
  }));
}
```

## 7. Grafana Dashboard Configuration

```yaml
# infrastructure/monitoring/grafana-dashboards/risk-monitoring.json
{
  "dashboard": {
    "title": "SRMP Risk Monitoring",
    "panels": [
      {
        "title": "Overall Risk Score",
        "type": "gauge",
        "targets": [
          {
            "expr": "srmp_risk_overall_score"
          }
        ],
        "thresholds": {
          "steps": [
            { "value": 0, "color": "green" },
            { "value": 50, "color": "yellow" },
            { "value": 75, "color": "red" }
          ]
        }
      },
      {
        "title": "Risk Breakdown",
        "type": "bar",
        "targets": [
          {
            "expr": "srmp_risk_breakdown{component=~\"wechat|ai|database|infrastructure|thirdparty\"}"
          }
        ]
      },
      {
        "title": "Active Alerts",
        "type": "stat",
        "targets": [
          {
            "expr": "count(srmp_risk_alerts{status=\"active\"})"
          }
        ]
      },
      {
        "title": "Service Health",
        "type": "table",
        "targets": [
          {
            "expr": "up{job=~\"redis|elasticsearch|postgres|rocketmq\"}"
          }
        ]
      },
      {
        "title": "Risk Trends (7 days)",
        "type": "graph",
        "targets": [
          {
            "expr": "srmp_risk_overall_score[7d]"
          }
        ]
      }
    ],
    "refresh": "30s",
    "time": {
      "from": "now-24h",
      "to": "now"
    }
  }
}
```

This comprehensive risk monitoring dashboard provides:

1. **Real-time Monitoring** - WebSocket updates every 30 seconds
2. **Automated Alerts** - Critical risks trigger immediate notifications
3. **Risk Scoring** - Quantified risk levels for each area
4. **Mitigation Tracking** - Progress on risk resolution
5. **Historical Trends** - Risk evolution over time
6. **Actionable Insights** - Specific recommendations for each risk
7. **Multi-channel Alerts** - WeChat, Email, and incident creation

The dashboard helps the team proactively manage risks throughout the SRMP development and deployment lifecycle.